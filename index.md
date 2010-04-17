---
title: Sprinkle Cheatsheet
---

Packages
--------

##### Options

- *requires* :ruby
- *recommends* :ansi_color
- *optional* :ansi_color
- *verify* { ... specify verifiers (see verifiers) ... }

##### Example

        # package :package_name, :provides => :virtual_package_name do
        #   ... installers, options, verifiers ...
        # end

        package :postgres, :provides => :database do
          source 'http://example.com/postgres.tgz'
          requires :build_essential
          verify { has_executable 'psql' }
        end

Verifiers
---------

- **has\_directory** "/path/to/dir"
  Tests if directory exists.

- **has\_executable** "executable\_name"
  Tests if global executable is available.

- **has\_executable** "/path/to/executable"
  Tests if executable exists at particular path.

- **has\_executable\_with\_version** "executable", "2.4.5", "-v"
  Tests if executable (1st arg) exists, then runs it with -v (3rd arg, default: -v) and matches "2.4.5" (2nd arg) in output.

- **has\_version\_in\_grep** "command", "2.4.5"
  Tests if output of any command (1st arg) matches given version (2nd arg).

- **has\_file** "/path/to/file"
  Tests if specific file exists.

- **file\_contains** "/path/to/file", "text"
  Tests if specific file contains particular text.

- **has\_process** "httpd"
  Tests if specific process is running.

- **ruby\_can\_load** "zlib", "readline", ...
  Tests if ruby can require specified libraries. Loads requires rubygems automatically.

- **has_gem** "haml", "3.0.0"
  Tests if gem list has specified gem with optional specified version (2nd arg, default: nil)

- **has_symlink** "/path/to/symlink", "/path/to/file"
  Tests if symlink exists, and optionally tests if symlink targets a specific file (2nd arg, default: nil)

##### Example

        package :postgres, :provides => :database do
          # ... installations, options ...
        
          verify do 
            has_executable 'psql'
            # ... more verifiers go here ...
          end
        end


Actors
------

- ### **capistrano**

  Uses capistrano to talk to remote computer.
    
  ##### Options

  - *recipes* "recipe_name" (default: "deploy")
    
- ### **local**
  
  Talks to local computer.
  
- ### **ssh**

  Uses ssh (directly) to talk to remote computer.
    
  ##### Options

  - *roles*
  - *gateway*
  - *user* (default: "root")
  - *password*
    
- ### **vlad**

  Uses vlad to talk to remote computer.
    
  ##### Options

  - script "deploy"
    

Deployment Options
------------------

        deployment do
          # Choose a delivery actor with native settings.
          # See: Actors
          #
          # delivery :actor do
          #   ... actor settings ...
          # end

          delivery :capistrano do
            recipes 'deploy'
          end
    
          # Specify installation options for certain installers.
          # (Normally only global options for source installer.)
          source do
            prefix    '/usr/local'
            archives  '/usr/local/sources'
            builds    '/usr/local/build'
          end
        end

Installers
----------

- ### **common**

  Base installer inherited in all installers. Provides pre/post install hooks.
  
  ##### Hooks

  - pre 'echo "foo"'  
    pre ['echo "foo"', 'echo "bar"']  
    pre { 'echo "foo"' }  
    pre { ['echo "foo"', 'echo "bar"'] }
  - post 'echo "foo"'  
    post ['echo "foo"', 'echo "bar"']  
    post { 'echo "foo"' }  
    post { ['echo "foo"', 'echo "bar"'] }

- ### **apt**

  Installs using apt-get
  
  ##### Options

  - *dependency_only* (default: false)

  ##### Example

        apt 'foo_package' { :dependencies_only => true }

- ### **binary**

  Installs a precompiled binary from archive.
  
  ##### Options
  - *prefix* "/usr/local"
  - *archives* "/usr/local/archives"

  ##### Example

        binary "http://some.url.com/archive.tar.gz" do
          prefix "/home/user/local"
          archives "/home/user/sources"
        end

- ### **gem**

  Runs gem install.
  
  ##### Options
  - *source* "http://gems.github.com" (adds --source)
  - *version* "2.4.5" (adds --version)
  - *repository* "foo" (adds --install-dir)
  - *build_docs* (removes "--no-rdoc --no-ri")
  - *http-proxy* (adds --http-proxy)
  - *build_flags* (adds build flags to the gem)
  
  ##### Example
  
        gem 'magic_beans_package' do
          source 'http://gems.github.com'
        end

- ### **push_text**

  Appends text to file.
  
  ##### Options

  - *sudo* true (default: false)
  
  ##### Example
  
        package :magic_beans do
          push_text 'magic_beans', '/etc/apache2/apache2.conf', :sudo => true
        end

- ### **rake**

  Runs a rake command.
  
  ##### Options

  - *rakefile* "/path/to/rakefile"
  
  ##### Example

        package :spec, :rakefile => "/var/setup/Rakefile" do
          rake 'spec'
        end

- ### **source**

  Downloads, extracts, configures, builds, installs from source.
  
  ##### Hooks
  - pre/post :prepare
  - pre/post :download
  - pre/post :extract
  - pre/post :configure
  - pre/post :build
  - pre/post :install

  ##### Options
  - *prefix* "/usr/local"
  - *builds* "/usr/local/builds"
  - *archives* "/usr/local/archives"
  - *custom_install* true (default: false)
  - *custom_archive* "foobar.tar.gz"
  
  ##### Example

        package :magic_beans do
          source 'http://magicbeansland.com/latest-1.1.1.tar.gz' do
            prefix '/usr/local'
            pre :prepare { 'echo "Here we go folks."' }
            post :extract { 'echo "I believe..."' }
            pre :build { 'echo "Cross your fingers!"' }
          end
        end

- ### **transfer**

  Pushes local files to remote servers, optionally rendering erb tags.

  ##### Options
  - *recursive* false (default: true if render => false)
  - *render* true (default: false)
  - *sudo* true (default: false)
  
  ##### Example
  
        package :nginx_conf do
          nginx_port = 8080
          transfer 'files/nginx.conf', '/etc/nginx.conf', :render => true, :sudo => true
        end