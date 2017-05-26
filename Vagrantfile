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
    "ami"            => 'unknown',
    "box"            => 'aws-dummy',
    "config"         => nil,
    "elasticIP"      => false,
    "ftpProxy"       => nil,
    "httpProxy"      => nil,
    "httpsProxy"     => nil,
    "iamRole"        => nil,
    "instanceType"   => 't2.micro',
    "keypair"        => 'ec2-keypair',
    "name"           => nil,
    "noProxy"        => 'localhost,127.0.0.1'
    "profile"        => 'default',
    "publicIP"       => false,
    "securityGroups" => '',
    "sshUser"        => 'ec2-user',
    "sshDir"         => '~/.ssh',
    "subnet"         => nil,
    "sudo"           => false,
    "userData"       => nil,
    "tags"           => {}
}

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
    [ '--elastic-ip',            GetoptLong::OPTIONAL_ARGUMENT ],
    [ '--ftp-proxy',             GetoptLong::OPTIONAL_ARGUMENT ],
    [ '--http-proxy',            GetoptLong::OPTIONAL_ARGUMENT ],
    [ '--https-proxy',           GetoptLong::OPTIONAL_ARGUMENT ],
    [ '--iam-role',              GetoptLong::OPTIONAL_ARGUMENT ],
    [ '--instance-type',         GetoptLong::OPTIONAL_ARGUMENT ],
    [ '--keypair',               GetoptLong::OPTIONAL_ARGUMENT ],
    [ '--name',                  GetoptLong::OPTIONAL_ARGUMENT ],
    [ '--no-proxy',              GetoptLong::OPTIONAL_ARGUMENT ],
    [ '--profile',               GetoptLong::OPTIONAL_ARGUMENT ],
    [ '--public-ip',             GetoptLong::OPTIONAL_ARGUMENT ],
    [ '--security-groups',       GetoptLong::OPTIONAL_ARGUMENT ],
    [ '--subnet',                GetoptLong::OPTIONAL_ARGUMENT ],
    [ '--ssh-user',              GetoptLong::OPTIONAL_ARGUMENT ],
    [ '--ssh-dir',               GetoptLong::OPTIONAL_ARGUMENT ],
    [ '--sudo',                  GetoptLong::OPTIONAL_ARGUMENT ],
    [ '--tags',                  GetoptLong::OPTIONAL_ARGUMENT ],
    [ '--user-data',             GetoptLong::OPTIONAL_ARGUMENT ]
)

# load from config file/s if exists, before we look at command line
if File.exist?( "./config.json" )
    userConfig = userConfig.merge( JSON.parse( FILE.read( "./config.json" ) ) )
end
if File.exist?( "./config.yml" )
    userConfig = userConfig.merge( YAML.load_file( "./config.yml" ) )
end

opts.each do |opt, arg|
    case opt
        when '--ami'
            userConfig[ "ami"            ] = arg
        when '--box'
            userConfig[ "box"            ] = arg
        when '--config'
            userConfig[ "config"         ] = File.expand_path( arg )
        when '--elastic-ip'
            userConfig[ "elasticIP"      ] = arg
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
        when '--profile'
            userConfig[ "profile"        ] = arg
        when '--public-ip'
            userConfig[ "publicIP"       ] = arg
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
        when '--tags'
            newTags = nil

            if arg.start_with?( 'file://' )
                newTags = YAML.load_file( File.expand_path( arg[  7, arg.length ] ) )
            elsif arg.start_with?( 'yamlfile://' )
                newTags = YAML.load_file( File.expand_path( arg[ 11, arg.length ] ) )
            elsif arg.start_with?( 'yaml://' )
                newTags = YAML.load( arg[ 7, arg.length ] )
            elsif arg.start_with?( 'jsonfile://' )
                newTags = JSON.parse( File.read( File.expand_path( arg[ 11, arg.length ] ) ) )
            elsif arg.start_with?( 'json://' )
                newTags = JSON.parse( arg[ 7, arg.length ] )
            else
                newTags = YAML.load( arg ) 
            end

            userConfig[ "tags" ] = userConfig[ "tags" ].merge( newTags )
        when '--user-data'
            if arg.start_with?( 'file://' )
                userConfig[ "userData" ] = File.read( File.expand_path( arg[ 7, arg.length ] ) )
            else
                userConfig[ "userData" ] = arg
            end
    end
end

# Check if a custom config passed in
unless userConfig[ "config" ].nil?

    newPath   = userConfig[ "config" ]
    newConfig = nil
    
    if newPath.start_with?( 'file://' )
        newConfig = YAML.load_file( File.expand_path( newPath[ 7, newPath.length ] ) )
    elsif newPath.start_with?( 'yamlfile://' )
        newConfig = YAML.load_file( File.expand_path( newPath[ 11, newPath.length ] ) )
    elsif newPath.start_with?( 'yaml://' )
        newConfig = YAML.load( newPath[ 7, arg.length ] )
    elsif newPath.start_with?( 'jsonfile://' )
        newConfig = JSON.parse( FILE.read( File.expand_path( newPath[ 11, newPath.length ] ) ) )
    elsif newPath.start_with?( 'json://' )
        newConfig = JSON.parse( newPath[ 7, newPath.length ] )
    else
        newConfig = YAML.load( newPath ) 
    end

    userConfig = userConfig.merge( newConfig )
end


############################################################################################
# MAIN
############################################################################################

VAGRANTFILE_API_VERSION = "2"

Vagrant.configure( VAGRANTFILE_API_VERSION ) do |config|

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

        mybox.vm.provider :aws do |aws, override|

            # ssh settings
            override.ssh.username = userConfig[ "sshUser" ]
        
            # Use a private key file based on the keypair name if possible
            private_key_path = File.expand_path( userConfig[ "sshDir"  ] + "/" + userConfig[ "keypair" ] + ".pem" )
            if File.exist?( private_key_path )
                override.ssh.private_key_path = private_key_path
            end

            # Instance settings
            aws.ami                       = userConfig[ "ami"            ]
            aws.associate_public_ip       = userConfig[ "publicIP"       ]
            aws.aws_profile               = userConfig[ "profile"        ]
            aws.iam_instance_profile_name = userConfig[ "iamRole"        ]
            aws.instance_type             = userConfig[ "instanceType"   ]
            aws.keypair_name              = userConfig[ "keypair"        ]
            aws.security_groups           = userConfig[ "securityGroups" ]
            aws.subnet_id                 = userConfig[ "subnet"         ]
            aws.user_data                 = userConfig[ "userData"       ]
            aws.elastic_ip                = userConfig[ "elasticIP"      ]
        
            # Having a Name tag has special meaning to the console, so we provide custom support
            tags = userConfig[ 'tags' ]
            unless tags.nil?
                tags = { 'Name' => userConfig[ 'name' ] }.merge( userConfig[ 'tags' ] )
            end
        
            aws.tags                      = tags
        
            # Always connect via private ip, ie through a vpn
            aws.ssh_host_attribute        = :private_ip_address

        end


        ##############################################################
        # Provision

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
