# -*- mode: ruby -*-
# vi: set ft=ruby :

require 'getoptlong'
require 'json'
require 'yaml'

require 'vagrant/util/downloader'

# Import inifile gem to parse ini files if its installed
inifile_found = Gem::Dependency.new( 'inifile' ).matching_specs.max_by(&:version)
if inifile_found
    require 'inifile'
end

# Import dotenv gem to parse env files if its installed
dotenv_found = Gem::Dependency.new( 'dotenv' ).matching_specs.max_by(&:version)
if dotenv_found
    require 'dotenv'
end


#########################################################################################
# DEFAULTS
#########################################################################################

# If copying this Vagrantfile, please modify these defaults for your own use case or use a
# config.(json|yml|ini|env) in the same directory

userConfig = {
    "ami"            => nil,
    "box"            => 'aws-dummy',
    "config"         => nil,
    "connectWith"    => "publicIP",
    "ebsOptimize"    => false,
    "elasticIP"      => false,
    "elb"            => nil,
    "ftpProxy"       => nil,
    "httpProxy"      => nil,
    "httpsProxy"     => nil,
    "iamRole"        => nil,
    "instanceType"   => 't2.micro',
    "keypair"        => 'ec2-keypair',
    "name"           => nil,
    "noProxy"        => 'localhost,127.0.0.1'
    "monitor"        => false,
    "profile"        => 'default',
    "protocol"       => 'ssh',
    "publicIP"       => false,
    "run"            => nil,
    "securityGroups" => '',
    "sshUser"        => 'ec2-user',
    "sshDir"         => '~/.ssh',
    "subnet"         => nil,
    "sudo"           => false,
    "tenancy"        => 'default',
    "userData"       => nil,
    "tags"           => {}
}

#########################################################################################

# Performs remote proxy configuration via the vagrant-proxyconf plugin
# Params:
# +config+:: The configuration we are working on
# +userConfig+:: +Hash+ object representing the configuration for the underlying instance
def run_proxy_config( config, userConfig = {} )
    
    if Vagrant.has_plugin?( "vagrant-proxyconf" )
	
        # Set proxy if one is defined and the plugin is installed
        unless userConfig[ "ftpProxy" ].nil?
            config.proxy.ftp      = userConfig[ "ftpProxy"   ]
        end
	
        unless userConfig[ "httpProxy" ].nil?
            config.proxy.http     = userConfig[ "httpProxy"  ]
        end
	
        unless userConfig[ "httpsProxy" ].nil?
            config.proxy.https    = userConfig[ "httpsProxy" ]
        end

        unless userConfig[ "noProxy" ].nil?
            config.proxy.no_proxy = userConfig[ "noProxy"    ]
        end
        
    end # END proxy plugin check
    
end

# Performs configuration shared across all providers
# Params:
# +config+:: The configuration we are working on
# +userConfig+:: +Hash+ object representing the configuration for the underlying instance
def run_core_config( config, userConfig = {} )
    config.vm.box = userConfig[ "box" ]
    
    # Update the underlying os type
    case userConfig[ "protocol" ]
        when 'winrm', 'winssh'
            config.vm.guest = :windows
        else
            config.vm.guest = :linux
    end
	
    # Default non provider specific connection settings
    case userConfig[ "protocol" ]
        when 'winrm'
            config.winrm.username = userConfig[ "winrmUser"     ]
            config.winrm.password = userConfig[ "winrmPassword" ]
        else
            config.ssh.username = userConfig[ "sshUser" ]
    end
    
end

def run_provider( mybox, userConfig = {} )
    %i( aws vbox esx ).each do |provider|
            
        # Will run the first matching provider only based on the box type
        case provider
            when :aws
                run_aws_provider(  mybox, userConfig )
            when :vbox
                run_vbox_provider( mybox, userConfig )
            when :esx
                run_esx_provider(  mybox, userConfig )
        end
            
    end
end

# Creates A Vmware ESX resource
# Params:
# +mybox+:: The box definition we are working on
# +userConfig+:: +Hash+ object representing the configuration for the underlying instance
def run_esx_provider( mybox, userConfig = {} )
    puts "Vmware provider is Not Implemented yet"
end

# Creates A Virtualbox resource
# Params:
# +mybox+:: The box definition we are working on
# +userConfig+:: +Hash+ object representing the configuration for the underlying instance
def run_vbox_provider( mybox, userConfig = {} )
    puts "Virtualbox provider is Not Implemented yet"
    
    mybox.vm.provider :virtualbox do |vbox, override|
        
        vbox.gui          = userConfig[ "gui" ]
        vbox.linked_clone = true
        
        vbox.customize ["modifyvm", :id, "--cpuexecutioncap", "50"]
        
        vbox.memory       = 2048
        vbox.cpus         = 2
        
    end  # END VBOX provider
end

# Creates An AWS resource
# Params:
# +mybox+:: The box definition we are working on
# +userConfig+:: +Hash+ object representing the configuration for the underlying instance
def run_aws_provider( mybox, userConfig = {} )
    
    # AWS specifics
    mybox.vm.provider :aws do |aws, override|

        # connection settings
        case userConfig[ "protocol" ]
            when 'winrm'
                # For AWS get encrypted password from the admin account
                override.winrm.username = "Administrator"
                override.winrm.password = :aws
            else
                override.ssh.username = userConfig[ "sshUser" ]
        end
        
        # Use a private key file based on the keypair name if possible
        private_key_path = File.expand_path( format( "%s.%s", File.join( userConfig[ "sshDir"  ], userConfig[ "keypair" ] ), "pem" ) )
        if File.exist?( private_key_path )
            override.ssh.private_key_path = private_key_path
        end

        # Instance settings
        aws.ami                       = userConfig[ "ami"            ]
        aws.associate_public_ip       = userConfig[ "publicIP"       ]
        aws.aws_profile               = userConfig[ "profile"        ]
        aws.ebs_optimized             = userConfig[ "ebsOptimize"    ]
        aws.elastic_ip                = userConfig[ "elasticIP"      ]
        aws.elb                       = userConfig[ "elb"            ]
        aws.iam_instance_profile_name = userConfig[ "iamRole"        ]
        aws.instance_type             = userConfig[ "instanceType"   ]
        aws.keypair_name              = userConfig[ "keypair"        ]
        aws.monitoring                = userConfig[ "monitor"        ]
        aws.security_groups           = userConfig[ "securityGroups" ]
        aws.subnet_id                 = userConfig[ "subnet"         ]
        aws.tenancy                   = userConfig[ "tenancy"        ]
        
        # Having a Name tag has special meaning to the console, so we provide custom support
        tags = userConfig[ 'tags' ]
        unless tags.nil?
            tags = { 'Name' => userConfig[ 'name' ] }.merge( userConfig[ 'tags' ] )
        end
        
        aws.tags                      = tags
        
        # Determine best way to connect, private ip address (i.e. through a vpn) or public etc
        case userConfig[ "connectWith" ]
            when "privateIP"
                aws.ssh_host_attribute        = :private_ip_address
            when "dns"
                aws.ssh_host_attribute        = :dns_name
            else
                aws.ssh_host_attribute        = :public_ip_address
        end
            
        user_data = userConfig[ "userData" ]
            
        # if winrm we need to ensure we enable it on first boot
        if userConfig[ "protocol" ] == 'winrm'
                
            user_data = <<-END.gsub( /^\s+\|/, '' )
              |<powershell>
              |  Enable-PSRemoting -Force
              |  netsh advfirewall firewall add rule name="WinRM HTTP" dir=in localport=5985 protocol=TCP action=allow
              |</powershell>
            END

        end
            
        aws.user_data = user_data

    end # END AWS provider
end

# Downloads a file from source to the target directory
# Params:
# +source+:: +String+ object of the ftp, http or https path to download from
# +download_dir+:: +String+ object set to the directory to download to
# +delete_if_exists+:: +Boolean+ object set to true if you want to overwrite existing files
def downloadFile( source, download_dir = ".", delete_if_exists = true )
    
    data = File.basename( source )
    
    # Create our path to download to
    download_path = File.join( download_dir, data )
        
    # Empty out any pre existing download as long as we dont override *this* script
    if ( data != "Vagrantfile" ) && delete_if_exists && download_path.file?
        download_path.delete
    end

    # Download
    Vagrant::Util::Downloader.new( source, download_path ).download!
        
    return download_path
end

# Standardizes reading data from a file or from a string
# Params:
# +data+:: +String+ object that specifies an optional type prefix (of the form yaml:// etc) and the data itself e.g. a file path
# +default_format+:: +String+ object indicating the format to interpret with if we cannot determine that directly from the data
# +download_dir+:: +String+ object set to the directory to download to if required
# +delete_if_exists+:: +Boolean+ object set to true if you want to overwrite existing files on download
def load_file( data, default_format = "yaml", download_dir = ".", delete_if_exists = true )
    
    res          = nil
    guess_file   = false
    
    # Add format if we know its a file and not a raw string
    if File.exist?( File.expand_path( data ) )
        guess_file = true
    elsif data.start_with?( 'file://' )
        guess_file = true
        data       = data[ 7, data.length ]
    elsif data.start_with?( 'ftp://', 'http://', 'https://' )
        guess_file = true
        data       = downloadFile( data, download_dir, true )
    end
    
    # we know its a file but not sure what..
    if guess_file
    
        data = File.expand_path( data )
        
        # check extension if that not clear use default, prefix the data
        if data.end_with?( ".yml", ".yaml" )
            data = "yamlfile://#{data}"
        elsif data.end_with?( ".json" )
            data = "jsonfile://#{data}"
        elsif data.end_with?( ".ini" )
            data = "inifile://#{data}"
        elsif data.end_with?( ".env" )
            data = "envfile://#{data}"
        elsif default_format in [ "yml", "yaml", "json", "ini", "env" ]
            # For yaml as it can be referred to with multiple extensions we standardize it
            data = "#{default_format}file://#{data}"
            data = data.sub( /^ymlfile\:\/\//, 'yamlfile://' )
        else
            data = "rawfile://#{data}"
        end
        
    end
    
    # Special handling for files that require gems as library might not be installed, default to raw
    if !inifile_found
        data = data.sub( /^inifile?\:\/\//, 'rawfile://' )
        data = data.sub( /^ini\:\/\//,      'raw://'     )
    end
    
    if !dotenv_found
        data = data.sub( /^envfile?\:\/\//, 'rawfile://' )
        data = data.sub( /^env\:\/\//,      'raw://'     )
    end

    # we should have correctly prefixed at this point
    if    data.start_with?( 'yamlfile://' )
        res = YAML.load_file( File.expand_path( data[ 11, data.length ] ) )
    elsif data.start_with?( 'yaml://' )
        res = YAML.load( data[ 7, data.length ] )
    elsif data.start_with?( 'jsonfile://' )
        res = JSON.parse( File.read( File.expand_path( data[ 11, data.length ] ) ) )
    elsif data.start_with?( 'json://' )
        res = JSON.parse( data[ 7, data.length ] )
    elsif data.start_with?( 'inifile://' )
        res = IniFile.load( File.expand_path( data[ 10, data.length ] ) ).to_h
    elsif data.start_with?( 'ini://' )
        res = IniFile.parse( data[ 6, data.length ] ).to_h
    elsif data.start_with?( 'envfile://' )
        # We dont necessarily want these loaded into the ENV variable so use this longer method
        res = Dotenv::Parser.call( File.read( File.expand_path( data[ 10, data.length ] ) ) )
    elsif data.start_with?( 'env://' )
        res = Dotenv::Parser.call( data[ 6, data.length ] )
    elsif data.start_with?( 'rawfile://' )
        res = File.read( File.expand_path( data[ 10, data.length ] ) )
    elsif data.start_with?( 'raw://' )
        res = data[ 6, data.length ]
    else
        res = data
    end
    
    return res
end


#########################################################################################

# Parameter handling
opts = GetoptLong.new(

    ##
    # Native Vagrant options, use to skip handling
    ##
    [ '--force',           '-f', GetoptLong::NO_ARGUMENT ],
    [ '--provision',       '-p', GetoptLong::NO_ARGUMENT ],
    [ '--provision-with',        GetoptLong::NO_ARGUMENT ],
    [ '--provider',              GetoptLong::NO_ARGUMENT ],
    [ '--help',            '-h', GetoptLong::NO_ARGUMENT ],
    [ '--check',                 GetoptLong::NO_ARGUMENT ],
    [ '--logout',                GetoptLong::NO_ARGUMENT ],
    [ '--token',                 GetoptLong::NO_ARGUMENT ],
    [ '--disable-http',          GetoptLong::NO_ARGUMENT ],
    [ '--http',                  GetoptLong::NO_ARGUMENT ],
    [ '--https',                 GetoptLong::NO_ARGUMENT ],
    [ '--ssh-no-password',       GetoptLong::NO_ARGUMENT ],
    [ '--ssh',                   GetoptLong::NO_ARGUMENT ],
    [ '--ssh-port',              GetoptLong::NO_ARGUMENT ],
    [ '--ssh-once',              GetoptLong::NO_ARGUMENT ],
    [ '--host',                  GetoptLong::NO_ARGUMENT ],
    [ '--entry-point',           GetoptLong::NO_ARGUMENT ],
    [ '--plugin-source',         GetoptLong::NO_ARGUMENT ],
    [ '--plugin-version',        GetoptLong::NO_ARGUMENT ],
    [ '--prune',                 GetoptLong::NO_ARGUMENT ],
    [ '--debug',                 GetoptLong::NO_ARGUMENT ],

    ##
    # Our Arguments
    ##
    [ '--ami',                   GetoptLong::OPTIONAL_ARGUMENT ],
    [ '--box',                   GetoptLong::OPTIONAL_ARGUMENT ],
    [ '--config',                GetoptLong::OPTIONAL_ARGUMENT ],
    [ '--connect-with',          GetoptLong::OPTIONAL_ARGUMENT ],
    [ '--ebs-optimize',          GetoptLong::OPTIONAL_ARGUMENT ],
    [ '--elastic-ip',            GetoptLong::OPTIONAL_ARGUMENT ],
    [ '--elb',                   GetoptLong::OPTIONAL_ARGUMENT ],
    [ '--ftp-proxy',             GetoptLong::OPTIONAL_ARGUMENT ],
    [ '--http-proxy',            GetoptLong::OPTIONAL_ARGUMENT ],
    [ '--https-proxy',           GetoptLong::OPTIONAL_ARGUMENT ],
    [ '--iam-role',              GetoptLong::OPTIONAL_ARGUMENT ],
    [ '--instance-type',         GetoptLong::OPTIONAL_ARGUMENT ],
    [ '--keypair',               GetoptLong::OPTIONAL_ARGUMENT ],
    [ '--name',                  GetoptLong::OPTIONAL_ARGUMENT ],
    [ '--no-proxy',              GetoptLong::OPTIONAL_ARGUMENT ],
    [ '--monitor',               GetoptLong::OPTIONAL_ARGUMENT ],
    [ '--profile',               GetoptLong::OPTIONAL_ARGUMENT ],
    [ '--protocol',              GetoptLong::OPTIONAL_ARGUMENT ],
    [ '--public-ip',             GetoptLong::OPTIONAL_ARGUMENT ],
    [ '--run',                   GetoptLong::OPTIONAL_ARGUMENT ],
    [ '--security-groups',       GetoptLong::OPTIONAL_ARGUMENT ],
    [ '--subnet',                GetoptLong::OPTIONAL_ARGUMENT ],
    [ '--ssh-user',              GetoptLong::OPTIONAL_ARGUMENT ],
    [ '--ssh-dir',               GetoptLong::OPTIONAL_ARGUMENT ],
    [ '--sudo',                  GetoptLong::OPTIONAL_ARGUMENT ],
    [ '--tags',                  GetoptLong::OPTIONAL_ARGUMENT ],
    [ '--tenancy',               GetoptLong::OPTIONAL_ARGUMENT ],
    [ '--user-data',             GetoptLong::OPTIONAL_ARGUMENT ]
)

# load from config file/s if exists, before we look at command line
# yml takes precedence, whereas subdirectories override
%w( . ./config ).each do |folder|

    confPath = nil
    
    # Go through supported config types for loading
    %w( env ini json yml yaml ).each do |extension|
        
        file_path = File.join( folder, "config.#{extension}" )
        
        if File.exist?( file_path )
            confPath = "#{extension}file://#{file_path}"
        end
        
    end
    
    # If a config file was found in the root, clean up the prefix before loading in its variables
    unless confPath.nil?
        confPath   = confPath.sub( /^ymlfile/, "yamlfile" )
        userConfig = userConfig.merge( load_file( confPath ) )
    end

end

opts.each do |opt, arg|
    case opt
        when '--config'
            userConfig = userConfig.merge( load_file( arg ) )
        when '--ami'
            userConfig[ "ami"            ] = arg
        when '--box'
            userConfig[ "box"            ] = arg
        when '--connect-with'
            userConfig[ "connectWith"    ] = arg
        when '--ebs-optimize'
            userConfig[ "ebsOptimize"    ] = arg
        when '--elastic-ip'
            userConfig[ "elasticIP"      ] = arg
        when '--elb'
            userConfig[ "elb"            ] = arg
        when '--ftp-proxy'
            userConfig[ "ftpProxy"       ] = arg
        when '--http-proxy'
            userConfig[ "httpProxy"      ] = arg
        when '--https-proxy'
            userConfig[ "httpsProxy"     ] = arg
        when '--iam-role'
            userConfig[ "iamRole"        ] = arg
        when '--instance-type'
            userConfig[ "instanceType"   ] = arg
        when '--keypair'
            userConfig[ "keypair"        ] = arg
        when '--name'
            userConfig[ "name"           ] = arg
        when '--no-proxy'
            userConfig[ "noProxy"        ] = arg
        when '--monitor'
            userConfig[ "monitor"        ] = arg
        when '--profile'
            userConfig[ "profile"        ] = arg
        when '--protocol'
            userConfig[ "protocol"       ] = arg
        when '--public-ip'
            userConfig[ "publicIP"       ] = arg
        when '--run'
            userConfig[ "run"            ] = File.expand_path( arg.sub( /^file\:\/\//, '' ) )
        when '--security-groups'
            userConfig[ "securityGroups" ] = arg.split( "," )
        when '--ssh-dir'
            userConfig[ "sshDir"         ] = arg
        when '--ssh-user'
            userConfig[ "sshUser"        ] = arg
        when '--subnet'
            userConfig[ "subnet"         ] = arg
        when '--sudo'
            userConfig[ "sudo"           ] = arg
        when '--tenancy'
            userConfig[ "tenancy"        ] = arg
        when '--tags'
            newTags = load_file( arg, "yaml" )

            userConfig[ "tags" ] = userConfig[ "tags" ].merge( newTags )
        when '--user-data'
            prefix = "raw://"
        
            # Adjust the prefix so its not interpreted as a yaml file etc
            if arg.start_with?( 'file://' )
                prefix = "raw"
            end
            
            userConfig[ "userData" ] = load_file( "#{prefix}#{arg}", "raw" )
    end
end


############################################################################################
# MAIN
############################################################################################

VAGRANTFILE_API_VERSION = "2"

Vagrant.configure( VAGRANTFILE_API_VERSION ) do |config|

    # Determine communication to use
    config.vm.communicator = userConfig[ "protocol" ]
    
    %w( sh ps1 bat ).each do |extension|
        
        Dir.glob( [
          File.join( ".", "deploy", "*.#{extension}" ),
          File.join( ".", "deploy.*.#{extension}" ),
          File.join( ".", "deploy.#{extension}" )
        ] ) do |script_path|
        
            if File.exist?( script_path )
    
                config.push.define "local-exec" do |push|
                    push.script = script_path
                end
            
            end
            
        end
    
    end
    
    %w( env ini json yml yaml ).each do |extension|
        
        Dir.glob( [
          File.join( ".", "deploy", "*.#{extension}" ),
          File.join( ".", "deploy.*.#{extension}" ),
          File.join( ".", "deploy.#{extension}" )
        ] ) do |script_path|
        
            if File.exist?( script_path )
    
                # sane defaults before merging in
                pushConfig = {
                    "host"        => nil,
                    "username"    => userConfig[ "sshUser" ],
                    "password"    => nil,
                    "secure"      => true,
                    "destination" => ".",
                    "source"      => "."
                }.merge( load_file( script_path ) )
        
                if pushConfig[ "cmd" ].nil?
            
                    # no custom cmd so assume s/ftp
                    config.push.define "ftp" do |push|
                        push.host        = pushConfig[ "host"        ]
                        push.username    = pushConfig[ "username"    ]
                        push.password    = pushConfig[ "password"    ]
                        push.secure      = pushConfig[ "secure"      ]
                        push.destination = pushConfig[ "destination" ]
                        push.source      = pushConfig[ "source"      ]
                        #passive
                        #exclude
                        #include
                    end
                
                else
                
                    # a cmd was specified, interpret it
                    config.push.define "local-exec" do |push|
                        push.inline = pushConfig[ "cmd" ] % pushConfig
                    end
                
                end # END cmd not specified
                
            end # END config exists
            
        end # END glob
    
    end # END extension loop
    
    
    ##############################################################
    # Proxy Setup
    run_proxy_config( config, userConfig )

    ##############################################################
    # Setup
    run_core_config( config, userConfig = {} )
    
    config.vm.define userConfig[ "name" ] do |mybox|
        
        ##############################################################
        # Configure
        run_provider( mybox, userConfig )


        ##############################################################
        # Provision
        
        if userConfig[ "run" ].nil?

            # No custom provisioner option passed in so check for file based existance
            
            # Keep track of which provision script we are matching
            matched_once  = false
            matched_every = false

            %w( sh ps1 bat ).each do |extension|

                script_path = File.join( ".", "provision", "shell", "run-once.#{extension}" )

                # Provision using a local shell script if one exists
                if File.exist?( script_path ) && !matched_once
                    mybox.vm.provision "shell", path: script_path,  privileged: userConfig[ "sudo" ]
                    matched_once = true
                end

                script_path = File.join( ".", "provision", "shell", "run-every.#{extension}" )

                if File.exist?( script_path ) && !matched_every
                    mybox.vm.provision "shell", path: script_path,  privileged: userConfig[ "sudo" ], run: "always"
                    matched_every = true
                end

            end # END shell provisioner determination

            # Reset
            matched_once  = false
            matched_every = false

            %w( pp ).each do |extension|

                script_path = File.join( ".", "provision", "shell", "run-once.#{extension}" )

                # Provision using puppet if exists
                if File.exist?( script_path ) && !matched_once

                    mybox.vm.provision :puppet do |puppet|
                        puppet.manifests_path = File.dirname(  script_path )
                        puppet.manifest_file  = File.basename( script_path )
                    end

                    matched_once = true
                end

                script_path = File.join( ".", "provision", "shell", "run-every.#{extension}" )

                if File.exist?( script_path ) && !matched_every

                    mybox.vm.provision :puppet, run: "always" do |puppet|
                        puppet.manifests_path = File.dirname(  script_path )
                        puppet.manifest_file  = File.basename( script_path )
                    end

                    matched_every = true
                end

            end # END puppet determination
            
            # Reset
            matched_once  = false
            matched_every = false

            %w( yml yaml ).each do |extension|

                script_path = File.join( ".", "provision", "shell", "run-once.#{extension}" )

                # Provision using ansible_local if exists
                if File.exist?( script_path ) && !matched_once

                    mybox.vm.provision "ansible_local" do |ansible|
                        ansible.playbook  = script_path
                        ansible.sudo      = userConfig[ "sudo" ]
                    end

                    matched_once = true
                end

                script_path = File.join( ".", "provision", "shell", "run-every.#{extension}" )

                if File.exist?( script_path ) && !matched_every

                    mybox.vm.provision "ansible_local", run: "always" do |ansible|
                        ansible.playbook  = script_path
                        ansible.sudo      = userConfig[ "sudo" ]
                    end

                    matched_every = true
                end

            end # END ansible provisioner determination

        else

            # A custom provisioner was passed in, see if we need to download it or just process it
            
            # Download the provisioning script if we have to
            if userConfig[ "run" ].start_with?( "ftp://", "http://", "https://" )
                userConfig[ "run" ] = downloadFile( userConfig[ "run" ], download_dir = "." )
            end
            
            # Should have downloaded but check perhpas if its a string literal
            if File.exist?( userConfig[ "run" ] )
                
                # Yml scripts generally indicate an ansible provisioner
                if userConfig[ "run" ].end_with?( ".yml", ".yaml" )

                    mybox.vm.provision "ansible_local", run: "always" do |ansible|
                        ansible.playbook  = userConfig[ "run"  ]
                        ansible.sudo      = userConfig[ "sudo" ]
                    end
                
                elsif userConfig[ "run" ].end_with?( ".pp" )

                    config.vm.provision :puppet do |puppet|
                        puppet.manifests_path = File.dirname(  userConfig[ "run" ] )
                        puppet.manifest_file  = File.basename( userConfig[ "run" ] )
                        #puppet.module_path = "puppet/modules"
                    end
                    
                elsif userConfig[ "run" ].end_with?( ".sh", ".bat", ".ps1" )
                    
                    # Shell scripts
                    mybox.vm.provision "shell", path: userConfig[ "run" ], privileged: userConfig[ "sudo" ], run: "always"
                    
                end # END provisioner extension check

            else
                
                # Run an inline shell command
                mybox.vm.provision "shell", inline: userConfig[ "run" ], privileged: userConfig[ "sudo" ], run: "always"
                
            end # END provisioner is a file check

        end # END custom provisioner check
    
    end # END define

end # END configure
