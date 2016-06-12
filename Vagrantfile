# -*- mode: ruby -*-
# # vi: set ft=ruby :


# http://blog.scottlowe.org/2014/10/22/multi-machine-vagrant-with-yaml/
require 'yaml'
require 'pp'


basedir = ENV.fetch('USERPROFILE', '')
basedir = ENV.fetch('HOME', '') if basedir == ''
basedir = basedir.gsub('\\', '/')
 
dir = File.expand_path(File.dirname(__FILE__))

# Read YAML file with nodes details
nodes = {}
nodes_yaml = "#{dir}/nodes.yaml"
if File.exists?(nodes_yaml)
  puts "Loading nodes '#{nodes_yaml}'"
  nodes = YAML.load_file(nodes_yaml)
else
  # TODO: throw an error
end
pp nodes

# Read YAML file with box details
configs = {}
boxes_yaml = "#{dir}/boxes.yaml"
if File.exists?(boxes_yaml)
  puts "Loading boxes configs '#{boxes_yaml}'"
  configs = YAML::load_file( boxes_yaml )  
else
    # TODO: throw an error
end
pp configs
 
box_config = {}
# Create boxes
Vagrant.configure('2') do |config| 
  nodes.each do |box|
    box_config = configs[box['box']]
    pp box_config
    image_name = box_config[:image_name]
    box_gui = box_config[:box_gui] != nil && box_config[:box_gui].to_s.match(/(true|t|yes|y|1)$/i) != nil
    box_cpus = box_config[:box_cpus].to_i
    box_memory = box_config[:box_memory].to_i
    newbox = box_config[:config_vm_newbox]
    image_filename = box_config[:image_filename]
    box_url = "file://#{basedir}/Downloads/#{image_filename}"
    config.vm.define box['name'] do |guest|
      guest.vm.box = image_name
      guest.vm.box_url = box_url
      guest.vm.network 'private_network', ip: box['ipaddress']
      guest.vm.provider :virtualbox do |vb|
        vb.name = box_name
        vb.memory = box_memory
      end      
      # Configure guest-specific port forwarding
      if config.vm.box !~ /windows/
        if config.vm.box =~ /centos/
          config.vm.network 'forwarded_port', guest: 8080, host: 8080, id: 'artifactory', auto_correct:true
        end
        config.vm.network 'forwarded_port', guest: 5901, host: 5901, id: 'vnc', auto_correct: true
        config.vm.host_name = 'linux.example.com'
        config.vm.hostname = 'linux.example.com'
      else
        # have to clear HTTP_PROXY to prevent
        # WinRM::WinRMHTTPTransportError: Bad HTTP response returned from server (503)
        # https://github.com/chef/knife-windows/issues/143
        ENV.delete('HTTP_PROXY')
        # NOTE: WPA dialog blocks chef solo and makes Vagrant fail on modern.ie box
        config.vm.communicator      = 'winrm'
        config.winrm.username       = 'vagrant'
        config.winrm.password       = 'vagrant'
        config.vm.guest             = :windows
        config.windows.halt_timeout = 15
        # Port forward WinRM and RDP
        config.vm.network :forwarded_port, guest: 3389, host: 3389, id: 'rdp', auto_correct: true
        config.vm.network :forwarded_port, guest: 5985, host: 5985, id: 'winrm', auto_correct:true
        config.vm.host_name         = 'windows7'
        config.vm.boot_timeout      = 120
        # Ensure that all networks are set to 'private'
        config.windows.set_work_network = true
        # on Windows, use default data_bags share
      end
    end
  end
end