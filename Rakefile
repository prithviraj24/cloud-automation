require 'rubygems'
require 'bundler/setup'
require 'json'
require 'aws-sdk-cloudformation'
require 'aws-sdk-ssm'
require 'dotenv'
require 'terminal-table'

Dotenv.load

class String
  def underscore
    self.gsub(/::/, '/').
    gsub(/([A-Z]+)([A-Z][a-z])/,'\1_\2').
    gsub(/([a-z\d])([A-Z])/,'\1_\2').
    tr("-", "_").
    downcase
  end
end

class Hash
  def transform_keys
    return enum_for(:transform_keys) { size } unless block_given?
    result = {}
    each_key do |key|
      result[yield(key)] = self[key]
    end
    result
  end
end

TEMPLATE_NAMES = Dir['templates/*'].collect{|d| File.basename(d).gsub(/\.(yaml|json)/, '')}.freeze
STACK_NAMES = {}.tap do |stack_names|
  TEMPLATE_NAMES.each do |template_name|
    stack_names[template_name] = Dir["template-parameters/*-#{template_name}.json"].collect{|d| File.basename(d).gsub('.json', '')}
    stack_names[template_name] << template_name if stack_names[template_name].empty?
  end
end.freeze
ENVIRONMENTS = ['staging', 'production']

desc 'List all templates in this repo'
task :list_templates do
  puts Terminal::Table.new headings: ['Template'], rows: TEMPLATE_NAMES.map { |i| [i] }
end

desc 'List all stacks in this repo'
task :list_stacks do
  rows = []
  STACK_NAMES.each do |template, stacks|
    stacks.each do |stack|
      rows << [stack, template]
    end
  end
  puts Terminal::Table.new headings: ['Stack', 'Template'], rows: rows
end

TEMPLATE_NAMES.each do |template_name|
  template_body =
    if File.exists?("templates/#{template_name}.yaml")
      File.read("templates/#{template_name}.yaml")
    end

  namespace template_name do
    desc "Validate the #{template_name} template"
    task :validate do
      cloudformation = Aws::CloudFormation::Client.new
      cloudformation.validate_template(template_body: template_body)
    end
  end

  STACK_NAMES[template_name].each do |stack_name|
    namespace stack_name do
      desc "To Create/Update the #{stack_name} stack with the #{template_name} template"
      task :update => "#{template_name}:validate" do
        # create cfn client
        cloudformation = Aws::CloudFormation::Client.new
        # find stack asked by user
        stack = cloudformation.list_stacks.stack_summaries.find {|stack| stack.stack_name == stack_name}

        stack_parameters =
          if File.exists?("template-parameters/#{stack_name}.json")
            JSON.parse(File.read("template-parameters/#{stack_name}.json")).
              map { |parameter_hash| parameter_hash.transform_keys(&:underscore) }
          end

        # To apply environment tag on stack.
        # Even applies to sub resources of stack that supports tagging.
        environment =
          if (substr_stack = stack_name.split('-').first) && ENVIRONMENTS.include?(substr_stack)
            substr_stack
          end

        # If stack does not exist or its in delete complete state. Create one.
        if !stack || stack.stack_status == 'DELETE_COMPLETE'
          STDERR.puts "*** Stack #{stack_name} does not exist, creating..."
          args = {
            stack_name:    stack_name,
            template_body: template_body,
            on_failure:    'DELETE',
            capabilities: ['CAPABILITY_IAM'],
            notification_arns: [
              ENV['CF_NOTIFY_TOPIC']
            ].compact
          }

          # Use stack if parameters are there, other it will take default params defined in template
          args[:parameters] = stack_parameters if stack_parameters
          args[:tags] = [{ key: "Environment", value: environment }] if environment
          cloudformation.create_stack(args)
          next
        end

        # Stack is already there
        STDERR.puts "*** Stack #{stack_name} already exists, updating..."
        args = {
          stack_name:     stack_name,
          template_body:  template_body,
          parameters:     stack_parameters,
          role_arn:       ENV['CF_ROLE'],
          capabilities:   ['CAPABILITY_IAM'],
          notification_arns: [
            ENV['CF_NOTIFY_TOPIC']
          ].compact
        }
        args[:tags] = [{ key: "Environment", value: environment }] if environment
        cloudformation.update_stack(args)
      end
    end
  end
end
