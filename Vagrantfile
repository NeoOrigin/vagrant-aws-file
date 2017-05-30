# -*- mode: ruby -*-
# vi: set ft=ruby :

require 'getoptlong'
require 'json'
require 'yaml'

#########################################################################################
# DEFAULTS
#########################################################################################

# If copying this Vagrantfile, please modify these defaults for your own use case or use a
# config.(json|yml) in the same directory

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

def load_file( data, default_format = "yaml" )
    res        = nil
    guess_file = false
    
    # Add format if we know its a file and not a raw string
    if File.exists?( File.expand_path( data ) )
        guess_file = true
    elsif data.start_with?( 'file://' )
        guess_file = true
        data = data[ 7, data.length ]
    end
    
    # we know its a file but not sure what..
    if guess_file
    
        data = File.expand_path( data )
        
        # check extension if that not clear use default
        if data.end_with?( ".yml" )
            data = "yamlfile://" + data
        elsif data.end_with?( ".yaml" )
            data = "yamlfile://" + data
        elsif data.end_with?( ".json" )
            data = "jsonfile://" + data
        elsif default_format == "yaml"
            data = "yamlfile://" + data
        elsif default_format == "json"
            data = "jsonfile://" + data
        else
            data = "rawfile://"  + data
        end
        
    end

    # we should have correctly prefixed at this point
    if data.start_with?( 'yamlfile://' )
        res = YAML.load_file( File.expand_path( data[ 11, data.length ] ) )
    elsif data.start_with?( 'yaml://' )
        res = YAML.load( data[ 7, data.length ] )
    elsif data.start_with?( 'jsonfile://' )
        res = JSON.parse( File.read( File.expand_path( data[ 11, data.length ] ) ) )
    elsif data.start_with?( 'json://' )
        res = JSON.parse( data[ 7, data.length ] )
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
[ ".", "./config" ].each do |folder|

    confPath = nil
    
    if File.exist?( folder + "/config.json" )
        confPath = "jsonfile://" + folder + "/config.json"
    end
    if File.exist?( folder + "/config.yml" )
        confPath = "yamlfile://" + folder + "/config.yml"
    end
    
    unless confPath.nil?
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
            userConfig[ "run"            ] = arg
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
            if arg.start_with?( 'file://' )
                prefix = "raw"
            else
                prefix = "raw://"
            end
            
            userConfig[ "userData" ] = load_file( prefix + arg, "raw" )
    end
end


############################################################################################
# MAIN
############################################################################################

VAGRANTFILE_API_VERSION = "2"

Vagrant.configure( VAGRANTFILE_API_VERSION ) do |config|

    # merge in .env files as a last resort
    if Vagrant.has_plugin?( "vagrant-env" )
        config.env.enable
        
        userConfig = userConfig.merge( ENV )
    end
    

    case userConfig[ "protocol" ]
    when 'winrm'
        config.vm.communicator = "winrm"
    else
        config.vm.communicator = "ssh"
    end
    ##############################################################
    # Proxy Setup

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
    end


    ##############################################################
    # Aws Setup

    config.vm.box = userConfig[ "box" ]
    
    config.vm.define userConfig[ "name" ] do |mybox|

        case userConfig[ "protocol" ]
        when 'winrm', 'winssh'
            config.vm.guest = :windows
        else
            config.vm.guest = :linux
        end
        
        mybox.vm.provider :aws do |aws, override|

            # connection settings
            case userConfig[ "protocol" ]
            when 'winrm'
                override.winrm.username = "Administrator"
                override.winrm.password = :aws
            else
                override.ssh.username = userConfig[ "sshUser" ]
            end
        
            # Use a private key file based on the keypair name if possible
            private_key_path = File.expand_path( userConfig[ "sshDir"  ] + "/" + userConfig[ "keypair" ] + ".pem" )
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
        
            # Always connect via private ip, ie through a vpn
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
            case userConfig[ "protocol" ]
            when 'winrm'
                
                user_data = <<-USERDATA
                  <powershell>
                    Enable-PSRemoting -Force
                    netsh advfirewall firewall add rule name="WinRM HTTP" dir=in localport=5985 protocol=TCP action=allow
                  </powershell>
                USERDATA
            end
            
            aws.user_data = user_data

        end


        ##############################################################
        # Provision
        
        unless userConfig[ "run" ].nil?
            if File.exist?( userConfig[ "run" ] )
                if userConfig[ "run" ].end_with( ".sh" )
                    mybox.vm.provision "shell", path: userConfig[ "run" ], privileged: userConfig[ "sudo" ], run: "always"
                elsif userConfig[ "run" ].end_with( ".yml" )
                    mybox.vm.provision "ansible_local", run: "always" do |ansible|
                        ansible.playbook  = userConfig[ "run" ]
                        ansible.sudo      = userConfig[ "sudo" ]
                    end
                 else
                     mybox.vm.provision "shell", inline: userConfig[ "run" ], privileged: userConfig[ "sudo" ], run: "always"
                end
            end
        end

        # Provision using a local shell script if one exists
        if File.exist?( "./provision/shell/run-once.sh" )
            mybox.vm.provision "shell", path: "./provision/shell/run-once.sh",  privileged: userConfig[ "sudo" ]
        end

        if File.exist?( "./provision/shell/run-every.sh" )
            mybox.vm.provision "shell", path: "./provision/shell/run-every.sh", privileged: userConfig[ "sudo" ], run: "always"
        end

        # Provision using ansible_local if exists
        if File.exist?( "./provision/ansible_local/run-once.yml" )

            mybox.vm.provision "ansible_local" do |ansible|
                ansible.playbook  = "./provision/ansible_local/run-once.yml"
                ansible.sudo      = userConfig[ "sudo" ]
            end

        end

        if File.exist?( "./provision/ansible_local/run-every.yml" )

            mybox.vm.provision "ansible_local", run: "always" do |ansible|
                ansible.playbook  = "./provision/ansible_local/run-every.yml"
                ansible.sudo      = userConfig[ "sudo" ]
            end

        end
    
    end

end
