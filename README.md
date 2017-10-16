# Ansible: Ruby on Rails Server (Ubuntu 16.x)

Use this ansible playbook to setup a  server with the following components:

* Nginx
* Puma App Server
* PostgreSQL
* Memcached
* Redis
* Sidekiq
* Monit (to keep Puma and Sidekiq runnig)
* Elasticsearch
* RVM
* Directories to deploy Rails with Capistrano and Puma App Server (see below)
* Swapfile
* Locales
* Tools (tmux, vim, htop, git, wget, curl etc.)

## Prerequisites & Config

1. create EC2 instance into AWS with ubuntu ami and download the key_pair.pem
1. cd ```hosts``` and modify the contents.
2. cd ```group_vars/all``` and modify the contents.

	There are a bunch of things you can set in ```group_vars/all```. Don't forget to add your host address to ```hosts```.

## Install Playbook

Run ```ansible-playbook site.yml -i hosts```.

## Rails Setup

This is just a loose guideline for what you need to deploy your app with this playbook and server config. Please keep in mind, that you need to modify some values depending on your setup (**especially passwords and paths!**)

### Gemfile

Add the following gems to your Gemfile and install via ```bundle install```:

```ruby
group :development do
	gem 'capistrano'
	gem 'capistrano3-puma'
	gem 'capistrano-rails', require: false
	gem 'capistrano-bundler', require: false
	gem 'capistrano-rvm'
	gem 'capistrano-sidekiq'
end
```

### Capfile

Add the following lines to your Capfile:

```ruby
# General
require 'capistrano/rails'
require 'capistrano/bundler'
require 'capistrano/rvm'
require 'capistrano/rails/assets'
require 'capistrano/rails/migrations'

# Puma
require 'capistrano/puma'
install_plugin Capistrano::Puma  # Default puma tasks
install_plugin Capistrano::Puma::Workers
install_plugin Capistrano::Puma::Monit  # if you need the monit tasks
install_plugin Capistrano::Puma::Nginx

# Sidekiq
require 'capistrano/sidekiq'
require 'capistrano/sidekiq/monit'
```

### config/deploy.rb


Your ```config/deploy.rb``` should look similar to this example:

```ruby
set :application, "app_name"
set :repo_url, "repo_url_app"
set :branch, "'branch_to_deploy'"
set :deploy_to, '/home/deploy/app_name'
set :linked_files, fetch(:linked_files, []).push('config/database.yml', 'config/puma.rb', 'config/secrets.yml')

# defines the RVM path to /usr/local/rvm
set :rvm_type, :system
set :rvm_ruby_version, 'ruby-2.3.1'
set :linked_dirs, fetch(:linked_dirs, []).push('log', 'tmp/pids', 'tmp/cache', 'tmp/sockets', 'vendor/bundle', 'public/system', 'public/uploads')
set :bundle_bins, fetch(:bundle_bins, []).push('sidekiq', 'sidekiqctl')
set :puma_conf, "#{shared_path}/config/puma.rb"
set :puma_init_active_record, true
```

### config/deploy/production.rb

Add the target host:

```ruby
server 'your_ip_address', user: 'deploy', roles: %w{app db web}
```
