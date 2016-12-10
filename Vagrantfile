# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|


  # TODO(hoatle): support this json configuration
  # config.omnibus.chef_version = :latest

  require 'json'
  load File.dirname(__FILE__) + '/lib/utility.rb'

  # Load default setting
  file = File.read(File.dirname(__FILE__) + '/vagrant_config.json')
  data_hash = JSON.parse(file)

  # Check and override if exist any match JSON object from vagrant_config_override.json
  if File.exist? (File.dirname(__FILE__) + '/vagrant_config_override.json')
    override_file = File.read(File.dirname(__FILE__) + '/vagrant_config_override.json')

    begin
      data_hash = overrides(data_hash, JSON.parse(override_file))
    rescue Exception => msg
      puts red(msg)
      puts red('from vagrant_config_override.json')
      ans = prompt yellow("some errors have occured and 'vagrant_config_override.json' file will not be used, do you want to continue? [y/n]: ")
      if ans.downcase != 'y'
        exit!
      end
    end

  end

  vm_hash = data_hash["vm"]

  # All Vagrant configuration is done here. The most common configuration
  # options are documented and commented below. For a complete reference,
  # please see the online documentation at vagrantup.com.

  # Every Vagrant virtual environment requires a box to build off of.
  config.vm.box = vm_hash["box"]

  # The url from where the 'config.vm.box' box will be fetched if it
  # doesn't already exist on the user's system.
  config.vm.box_url = vm_hash['box_url']

  # Other configs: https://docs.vagrantup.com/v2/vagrantfile/machine_settings.html

  if !vm_hash["box_version"].nil? and !vm_hash["box_version"].strip().empty?
    config.vm.box_version = vm_hash['box_version']
  end

  if !vm_hash["boot_timeout"].nil?
    config.vm.boot_timeout = vm_hash['boot_timeout']
  end

  if !vm_hash["box_check_update"].nil?
    config.vm.box_check_update = vm_hash['box_check_update']
  end

  if !vm_hash["box_download_checksum"].nil? and !vm_hash["box_download_checksum"].strip().empty?
    config.vm.box_download_checksum = vm_hash['box_download_checksum']

    # box_download_checksum_type must be specified if box_download_checksum is specified
    config.vm.box_download_checksum_type = vm_hash['box_download_checksum_type']
  end

  if !vm_hash["box_download_client_cert"].nil? and !vm_hash["box_download_client_cert"].strip().empty?
    config.vm.box_download_client_cert = vm_hash['box_download_client_cert']
  end

  if !vm_hash["box_download_ca_cert"].nil? and !vm_hash["box_download_ca_cert"].strip().empty?
    config.vm.box_download_ca_cert = vm_hash['box_download_ca_cert']
  end

  if !vm_hash["box_download_ca_path"].nil? and !vm_hash["box_download_ca_path"].strip().empty?
    config.vm.box_download_ca_path = vm_hash['box_download_ca_path']
  end

  if !vm_hash["box_download_insecure"].nil?
    config.vm.box_download_insecure = vm_hash['box_download_insecure']
  end

  if !vm_hash["communicator"].nil? and !vm_hash["communicator"].strip().empty?
    config.vm.communicator = vm_hash['communicator']
  end

  if !vm_hash["graceful_halt_timeout"].nil?
    config.vm.graceful_halt_timeout = vm_hash['graceful_halt_timeout']
  end

  if !vm_hash["guest"].nil? and !vm_hash["guest"].strip().empty?
    config.vm.guest = vm_hash['guest']
  end

  if !vm_hash["hostname"].nil? and !vm_hash["hostname"].strip().empty?
    config.vm.hostname = vm_hash['hostname']
  end

  if !vm_hash["post_up_message"].nil? and !vm_hash["post_up_message"].strip().empty?
    config.vm.post_up_message = vm_hash['post_up_message']
  end

  if !vm_hash["usable_port_range"].nil? and !vm_hash["usable_port_range"].strip().empty?
    ranges = vm_hash['usable_port_range'].split('..').map{|d| Integer(d)}
    config.vm.usable_port_range = ranges[0]..ranges[1]
  end

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  vm_networks = vm_hash['networks']
  vm_networks.each do |vm_network|
    if vm_network['mode'] == 'forwarded_port'
      vm_network['forwarded_ports'].each do |item|
        config.vm.network :forwarded_port, guest: item['guest'], host: item['host']
      end
    else
      options = {}
      case vm_network['mode']
      when 'private_network'
        options[:ip] = vm_network['ip'] unless vm_network['ip'].nil? or vm_network['ip'].strip().empty?
        if options[:ip].nil? or options[:ip].empty?
          # make `type: 'dhcp'` default when `ip` is not defined (nil or empty)
          options[:type] = 'dhcp'
        else
          options[:auto_config] = !(vm_network['auto_config'] == false)
        end
      when 'public_network'
        options[:ip] = vm_network['ip'] unless vm_network['ip'].nil? or vm_network['ip'].strip().empty?
        options[:bridge] = vm_network['bridge'] unless vm_network['bridge'].nil? or vm_network['bridge'].empty?
      end

      config.vm.network vm_network['mode'], options

    end
  end


  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  # config.vm.network :private_network, ip: "192.168.33.10"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network :public_network

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  # config.vm.synced_folder "../data", "/vagrant_data"

  vm_hash['synced_folders'].each do |item|
    options = {}
    host_os = Vagrant::Util::Platform.platform
    host_os_type = ''

    case host_os
    when /^(mswin|mingw).*/
      host_os_type = 'windows'
    when /^(linux|cygwin).*/
      host_os_type = 'linux'
    when /^(mac|darwin).*/
      host_os_type = 'mac'
    end

    # options from http://docs.vagrantup.com/v2/synced-folders/basic_usage.html
    options[:create] = item['create'] unless item['create'].nil?
    options[:disabled] = item['disabled'] unless item['disabled'].nil?
    options[:owner] = item['owner'] unless item['owner'].nil?
    options[:group] = item['group'] unless item['group'].nil?
    options[:mount_options] = item['mount_options'] unless item['mount_options'].nil?
    options[:type] = item['type'] unless item['type'].nil? or item['type'] == 'virtual_box'

    case item['type']
    when 'nfs'
      options[:nfs_export] = item['nfs_export'] if !!item['nfs_export'] == item['nfs_export']
      options[:nfs_udp] = item['nfs_udp'] if !!item['nfs_udp'] == item['nfs_udp']
      options[:nfs_version] = item['nfs_version'] unless item['nfs_version'].nil?
    when 'rsync'
      options[:rsync__args] = item['rsync__args'] unless item['rsync__args'].nil? or item['rsync__args'].strip().empty?
      options[:rsync__auto] = item['rsync__auto'] if !!item['rsync__auto'] == item['rsync__auto']
      options[:rsync__chown] = item['rsync__chown'] if !!item['rsync__chown'] == item['rsync__chown']
      options[:rsync__exclude] = item['rsync__exclude'] unless item['rsync__exclude'].nil? or item['rsync__exclude'].empty?
    when 'smb'
      options[:smb_host] = item['smb_host'] unless item['smb_host'].nil? or item['smb_host'].empty?
      options[:smb_password] = item['smb_password'] unless item['smb_password'].nil? or item['smb_password'].empty?
      options[:smb_username] = item['smb_username'] unless item['smb_password'].nil? or item['smb_password'].empty?
    end

    if item['supports'].nil? or item['supports'].include?(host_os_type)
      config.vm.synced_folder item['host'], item['guest'], options
    end
  end

  # plugins config
  plugins_hash = data_hash['plugins']

  puts config.methods(true)

  plugins_hash.each do |plugin_name, plugin_value|
    if plugin_value['required'] == true
      unless Vagrant.has_plugin?(plugin_name)
        puts red("required: '$ vagrant plugin install #{plugin_name}'")
        exit!
      end
    end

    if plugin_value.key?('config_key')
      config_key = plugin_value['config_key']
      if Vagrant.has_plugin?(plugin_name) and !config_key.nil? and !config_key.empty?
        puts red(config[config_key.to_sym])
        # TODO(hoatle): remove config_key and required keys?
        #config.instance_variable_set("@#{config_key}", plugin_value)
        # new_config = Vagrant::Config::V2::Root.new({
        #   config_key => plugin_value
        # })
        # config.merge(config, new_config)
        config.gatling.rsync_on_startup = false

        puts red(config.instance_variable_get(:@gatling))

        puts config.inspect
      end
    end
  end

  # ssh configuration
  config.ssh.forward_agent = vm_hash['forward_agent']

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  # View the documentation for the provider you're using for more
  # information on available options.
  config.vm.provider :virtualbox do |vb|

    vb_hash = data_hash['vb']

    # Don't boot with headless mode
    if vb_hash['gui']  == true
      vb.gui = true
    end

    # general settings for modifyvm: https://www.virtualbox.org/manual/ch08.html#vboxmanage-modifyvm
    # TODO(hoatle): add support for key<1-N> type
    # TODO(hoatle): add support for other settings

    # FIXME(hoatle): there were 3 loops here, why?
    # puts vb_hash

    general_settings_keys = ['name', 'groups', 'description', 'ostype', 'memory', 'vram', 'acpi',
      'ioapic', 'hardwareuuid', 'cpus', 'rtcuseutc', 'cpuhotplug', 'plugcpu', 'unplugcpu',
      'cpuexecutioncap', 'pae', 'longmode', 'synthcpu', 'hpet', 'hwvirtex', 'triplefaultreset',
      'nestedpaging', 'largepages', 'vtxvpid', 'vtxux', 'accelerate3d', 'bioslogofadein',
      'bioslogodisplaytime', 'bioslogoimagepath', 'biosbootmenu', 'snapshotfolder', 'firmware',
      'guestmemoryballoon', 'defaultfrontend'
    ]

    vb_hash.each do |key, val|
      if general_settings_keys.include?(key) and !vb_hash[key].nil?
        val = val.to_s.strip()
        if !val.empty?
          vb.customize ["modifyvm", :id, "--" + key, val]
        end
      end
    end

  end

  # Enable provisioning with chef solo, specifying a cookbooks path, roles
  # path, and data_bags path (all relative to this Vagrantfile), and adding
  # some recipes and/or roles.
  #
  # see: https://www.vagrantup.com/docs/provisioning/chef_solo.html
  chef_hash = data_hash['chef']

  if !chef_hash.nil? and chef_hash['enabled']
    config.vm.provision "chef_solo" do |chef|
      chef.log_level = chef_hash['log_level']
      chef.cookbooks_path = chef_hash['cookbooks_path']
      chef.data_bags_path = chef_hash['data_bags_path']
      chef.environments_path = chef_hash['environments_path']
      chef.environment = chef_hash['environment']
      chef.nodes_path = chef_hash['nodes_path']
      chef.recipe_url = chef_hash['recipe_url']
      chef.roles_path = chef_hash['roles_path']
      chef.synced_folder_type = chef_hash['synced_folder_type']

      unless chef_hash['roles'].nil?
        chef_hash['roles'].each do |role|
          chef.add_role role
        end
      end

      unless chef_hash['recipes'].nil?
        chef_hash['recipes'].each do |recipe|
          chef.add_recipe recipe
        end
      end

      chef.json = chef_hash['json']
    end
  end

  $display_ip_address = <<IP_ADDRESS
ipaddress=`hostname -I | cut -d' ' -f2`
echo "ip address: $ipaddress"
IP_ADDRESS

  config.vm.provision "shell", inline: $display_ip_address
  # Enable provisioning with chef server, specifying the chef server URL,
  # and the path to the validation key (relative to this Vagrantfile).
  #
  # The Opscode Platform uses HTTPS. Substitute your organization for
  # ORGNAME in the URL and validation key.
  #
  # If you have your own Chef Server, use the appropriate URL, which may be
  # HTTP instead of HTTPS depending on your configuration. Also change the
  # validation key to validation.pem.
  #
  # config.vm.provision :chef_client do |chef|
  #   chef.chef_server_url = ENV['KNIFE_CHEF_SERVER']
  #   chef.validation_key_path = "#{ENV['KNIFE_VALIDATION_KEY_FOLDER']}/#{ENV['OPSCODE_ORGNAME']}-validator.pem"
  #   chef.validation_client_name = "#{ENV['OPSCODE_ORGNAME']}-validator"
  #   chef.node_name = "#{ENV['OPSCODE_USER']}-vagrant"
  #   chef.run_list = [
  #     'motd',
  #     'minitest-handler'
  #   ]
  # end
  #
  # If you're using the Opscode platform, your validator client is
  # ORGNAME-validator, replacing ORGNAME with your organization name.
  #
  # If you have your own Chef Server, the default validation client name is
  # chef-validator, unless you changed the configuration.
  #
  #   chef.validation_client_name = "ORGNAME-validator"

end
