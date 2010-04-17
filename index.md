---
title: Sprinkle Cheatsheet
---

Packages
--------

Options

- *requires* :ruby
- *recommends* :ansi_color
- *optional* :ansi_color

Example

    # package :package_name, :provides => :virtual_package_name do
    #   ... installers, options, verifiers ...
    # end

    package :postgres, :provides => :database do
      source 'http://example.com/postgres.tgz'
      requires :build_essential
      verify { has_executable 'psql' }
    end

Actors
------

- ### **capistrano**
  
  Uses capistrano to talk to remote computer.
    
    Options
    - *recipes* "recipe_name" (default: "deploy")
    
- ### **local**
  
  Talks to local computer.
  
- ### **ssh**

  Uses ssh (directly) to talk to remote computer.
    
    Options
    - *roles*
    - *gateway*
    - *user* (default: "root")
    - *password*
    
- ### **vlad**

  Uses vlad to talk to remote computer.
    
    Options
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
  
  Hooks

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
  
  Options
  - *dependency_only* (default: false)

  Example

        apt 'foo_package' { :dependencies_only => true }

- ### **binary**

  Installs a precompiled binary from archive.
  
  Options
  - *prefix* "/usr/local"
  - *archives* "/usr/local/archives"

  Example

        binary "http://some.url.com/archive.tar.gz" do
          prefix "/home/user/local"
          archives "/home/user/sources"
        end

- ### **gem**

  Runs gem install.
  
  Options
  - *source* "http://gems.github.com" (adds --source)
  - *version* "2.4.5" (adds --version)
  - *repository* "foo" (adds --install-dir)
  - *build_docs* (removes "--no-rdoc --no-ri")
  - *http-proxy* (adds --http-proxy)
  - *build_flags* (adds build flags to the gem)
  
  Example
  
        gem 'magic_beans_package' do
          source 'http://gems.github.com'
        end

- ### **push_text**

  Appends text to file.
  
  Options
  - *sudo* true (default: false)
  
  Example
  
        package :magic_beans do
          push_text 'magic_beans', '/etc/apache2/apache2.conf', :sudo => true
        end

- ### **rake**

  Runs a rake command.
  
  Options
  - *rakefile* "/path/to/rakefile"
  
  Example

        package :spec, :rakefile => "/var/setup/Rakefile" do
          rake 'spec'
        end

- ### **source**

  Downloads, extracts, configures, builds, installs from source.
  
  Hooks
  - pre/post :prepare
  - pre/post :download
  - pre/post :extract
  - pre/post :configure
  - pre/post :build
  - pre/post :install

  Options
  - *prefix* "/usr/local"
  - *builds* "/usr/local/builds"
  - *archives* "/usr/local/archives"
  - *custom_install* true (default: false)
  - *custom_archive* "foobar.tar.gz"
  
  Example

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

  Options
  - *recursive* false (default: true if render => false)
  - *render* true (default: false)
  - *sudo* true (default: false)
  
  Example
  
        package :nginx_conf do
          nginx_port = 8080
          transfer 'files/nginx.conf', '/etc/nginx.conf', :render => true, :sudo => true
        end