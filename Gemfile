source ENV['GEM_SOURCE'] || 'https://rubygems.org'

# Determines what type of gem is requested based on place_or_version.
def gem_type(place_or_version)
  if place_or_version =~ /^git:/
    :git
  elsif place_or_version =~ /^file:/
    :file
  else
    :gem
  end
end

# Find a location or specific version for a gem. place_or_version can be a
# version, which is most often used. It can also be git, which is specified as
# `git://somewhere.git#branch`. You can also use a file source location, which
# is specified as `file://some/location/on/disk`.
def location_for(place_or_version, fake_version = nil)
  if place_or_version =~ /^(git[:@][^#]*)#(.*)/
    [fake_version, { :git => $1, :branch => $2, :require => false }].compact
  elsif place_or_version =~ /^file:\/\/(.*)/
    ['>= 0', { :path => File.expand_path($1), :require => false }]
  else
    [place_or_version, { :require => false }]
  end
end

group :development, :test do
  gem 'rake',                    :require => false
  gem 'mocha', '~>0.10.5',       :require => false
  gem 'puppetlabs_spec_helper',  :require => false
  gem 'puppet-lint',             :require => false
  gem 'rspec', '~>2.14.0',       :require => false
end

beaker_version = ENV['BEAKER_VERSION'] || '~> 2.2'
group :system_tests do
  if beaker_version
    gem 'beaker', *location_for(beaker_version)
  else
    gem 'beaker',                :require => false, :platforms => :ruby
  end
  gem 'beaker-puppet_install_helper', :require => false
end

# The recommendation is for PROJECT_GEM_VERSION, although there are older ways
# of referencing these. Add them all for compatibility reasons. We'll remove
# later when no issues are known. We'll prefer them in the right order.
puppetversion = ENV['PUPPET_GEM_VERSION'] || ENV['GEM_PUPPET_VERSION'] || ENV['PUPPET_LOCATION'] || '>= 0'
gem 'puppet', *location_for(puppetversion)

# Only explicitly specify Facter/Hiera if a version has been specified.
# Otherwise it can lead to strange bundler behavior. If you are seeing this
# behavior, set `DEBUG_RESOLVER` environment variable to `1` and run bundle
# install.
facterversion = ENV['FACTER_GEM_VERSION'] || ENV['GEM_FACTER_VERSION'] || ENV['FACTER_LOCATION']
gem "facter", *location_for(facterversion) if facterversion
hieraversion = ENV['HIERA_GEM_VERSION'] || ENV['GEM_HIERA_VERSION'] || ENV['HIERA_LOCATION']
gem "hiera", *location_for(hieraversion) if hieraversion


# For Windows dependencies, these could be required based on the version of
# Puppet you are requiring. Anything greater than v3.5.0 is going to have
# Windows-specific dependencies dictated by the gem itself. The other scenario
# is when you are faking out Puppet to use a local file path / git path.
explicitly_require_windows_gems = false
puppet_gem_location = gem_type(puppetversion)
# This is not a perfect answer to the version check
if puppet_gem_location != :gem || puppetversion < '3.5.0'
  is_x64 = Gem::Platform.local.cpu == 'x64'
  if is_x64
    platform(:x64_mingw) do
      explicitly_require_windows_gems = true
    end
  else
    platform (:mingw) do
      explicitly_require_windows_gems = true
    end
  end
end

if explicitly_require_windows_gems
  gem "ffi", "~> 1.9.0", :require => false
  gem "win32-dir", "~> 0.3", :require => false
  gem "win32-eventlog", "~> 0.5", :require => false
  gem "win32-process", "~> 0.6", :require => false
  gem "win32-security", "~> 0.1", :require => false
  gem "win32-service", "~> 0.7", :require => false
  gem "minitar", "~> 0.5.4", :require => false

  if RUBY_VERSION =~ /^1\./ then
    gem "win32console", :require => false
  end

  # Puppet less than 3.7.0 requires these.
  # Puppet 3.5.0+ will control the actual requirements.
  gem "sys-admin", "~> 1.5", :require => false
  gem "win32-api", "~> 1.4.8", :require => false
  gem "win32-taskscheduler", "~> 0.2", :require => false
  gem "windows-api", "~> 0.4", :require => false
  gem "windows-pr", "~> 1.2", :require => false
end

if File.exists? "#{__FILE__}.local"
  eval(File.read("#{__FILE__}.local"), binding)
end

# vim:ft=ruby
