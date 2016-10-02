# macOS — YOU MUST use ruby 2.2.2 — rvm use ruby-2.2.2 (and then MAC=1 rake package:mac) 

# For Bundler.with_clean_env
require 'bundler/setup'

PACKAGE_NAME = "fpm"
TRAVELING_RUBY_VERSION = "2.3.1"
TRAVELING_RUBY_VERSION_OSX = "20150715-2.2.2"

if ENV['MAC']
  VERSION = "1.8.1-#{TRAVELING_RUBY_VERSION_OSX}"
else
  VERSION = "1.8.1-#{TRAVELING_RUBY_VERSION}"
end

FFI = "ffi-1.9.10"

desc "Package your app"
# task :package => ['package:linux:x86', 'package:linux:x86_64', 'package:mac']
task :package => ['package:linux:x86', 'package:linux:x86_64']
# task :package => ['package:mac']

namespace :package do
  namespace :linux do
    desc "Package your app for Linux x86"
    task :x86 => [:bundle_install,
                  "packaging/traveling-ruby-#{TRAVELING_RUBY_VERSION}-linux-x86.tar.xz",
                  "packaging/#{FFI}-#{TRAVELING_RUBY_VERSION}-linux-x86.tar.xz",
    ] do
      create_package("linux-x86")
    end

    desc "Package your app for Linux x86_64"
    task :x86_64 => [:bundle_install,
                     "packaging/traveling-ruby-#{TRAVELING_RUBY_VERSION}-linux-x86_64.tar.xz",
                     "packaging/#{FFI}-#{TRAVELING_RUBY_VERSION}-linux-x86_64.tar.xz",
    ] do
      create_package("linux-x86_64")
    end
  end

  desc "Package your app for macOS"
  task :mac => [:bundle_install, "packaging/traveling-ruby-#{TRAVELING_RUBY_VERSION_OSX}-osx.tar.gz"] do
    create_package("mac")
  end

  desc "Install gems to local directory"
  task :bundle_install do
    sh "rm -rf packaging/tmp"
    sh "mkdir -p packaging/tmp"
    sh "cp Gemfile Gemfile.lock fpm.gemspec packaging/tmp/"
    sh "mkdir -p packaging/tmp/lib/fpm"
    sh "cp lib/fpm/version.rb packaging/tmp/lib/fpm/version.rb"
    Bundler.with_clean_env do
      sh "cd packaging/tmp && env BUNDLE_IGNORE_CONFIG=1 bundle install --path ../vendor --without development"
    end
    sh "rm -rf packaging/tmp"
    sh "rm -f packaging/vendor/*/*/cache/*"

    if !ENV['MAC']
      sh "rm -rf packaging/vendor/ruby/*/extensions"
      sh "find packaging/vendor/ruby/*/gems -name '*.so' | xargs rm -f"
      sh "find packaging/vendor/ruby/*/gems -name '*.bundle' | xargs rm -f"
      sh "find packaging/vendor/ruby/*/gems -name '*.o' | xargs rm -f"
    end
  end
end

file "packaging/traveling-ruby-#{TRAVELING_RUBY_VERSION}-linux-x86.tar.xz" do
  download_runtime_github("linux-x86")
end

file "packaging/traveling-ruby-#{TRAVELING_RUBY_VERSION}-linux-x86_64.tar.xz" do
  download_runtime_github("linux-x86_64")
end

file "packaging/#{FFI}-#{TRAVELING_RUBY_VERSION}-linux-x86.tar.xz" do
  download_native_extension(FFI, "linux-x86")
end

file "packaging/#{FFI}-#{TRAVELING_RUBY_VERSION}-linux-x86_64.tar.xz" do
  download_native_extension(FFI, "linux-x86_64")
end

file "packaging/traveling-ruby-#{TRAVELING_RUBY_VERSION_OSX}-osx.tar.gz" do
  download_runtime("osx")
end

def create_package(target)
  package_name = "#{PACKAGE_NAME}-#{VERSION}-#{target}"
  package_dir = "dist/#{package_name}"
  sh "rm -rf #{package_dir}"
  sh "mkdir -p #{package_dir}"
  sh "mkdir -p #{package_dir}/lib/app"

  sh "cp -R lib/ #{package_dir}/lib/app/lib/"
  sh "cp -R bin/ #{package_dir}/lib/app/bin/"
  sh "cp -R templates/ #{package_dir}/lib/app/templates/"
  sh "cp -R misc/ #{package_dir}/lib/app/misc/"

  sh "mkdir #{package_dir}/lib/ruby"
  if ENV['MAC']
    sh "tar -xzf packaging/traveling-ruby-#{TRAVELING_RUBY_VERSION_OSX}-osx.tar.gz -C #{package_dir}/lib/ruby"
  else
    sh "tar -xzf packaging/traveling-ruby-#{TRAVELING_RUBY_VERSION}-#{target}.tar.xz -C #{package_dir}/lib/ruby"
  end

  sh "cp packaging/wrapper.sh #{package_dir}/fpm"
  sh "chmod +x #{package_dir}/fpm"
  sh "cp -pR packaging/vendor #{package_dir}/lib/"

  sh "cp Gemfile Gemfile.lock fpm.gemspec #{package_dir}/lib/vendor/"
  sh "mkdir -p #{package_dir}/lib/vendor/lib/fpm"
  sh "cp lib/fpm/version.rb #{package_dir}/lib/vendor/lib/fpm/version.rb"

  sh "mkdir #{package_dir}/lib/vendor/.bundle"
  sh "cp packaging/bundler-config #{package_dir}/lib/vendor/.bundle/config"

  if !ENV['MAC']
    sh "tar -xzf packaging/#{FFI}-#{TRAVELING_RUBY_VERSION}-#{target}.tar.xz -C #{package_dir}/lib/vendor/ruby"
  end

  if !ENV['DIR_ONLY']
    # sh "cd dist && tar -cf - #{package_name}/ | xz --x86 --compress --force -9 --extreme - > #{package_name}.tar.xz"
    sh "rm -f #{package_name}.7z"
    sh "cd dist/#{package_name} && 7za a -m0=lzma2 -mx=9 -mfb=64 -md=64m -ms=on ../#{package_name}.7z ."
  end
end

def download_runtime(target)
  sh "cd packaging && curl -L -O --fail " +
         "https://d6r77u77i8pq3.cloudfront.net/releases/traveling-ruby-#{TRAVELING_RUBY_VERSION_OSX}-#{target}.tar.gz"
end

def download_runtime_github(target)
  sh "cd packaging && curl -L -O --fail " +
         "https://github.com/develar/traveling-ruby/releases/download/v#{TRAVELING_RUBY_VERSION}/traveling-ruby-#{TRAVELING_RUBY_VERSION}-#{target}.tar.xz"
end

def download_native_extension(gem_name_and_version, target)
  sh "cd packaging && curl -L -O --fail " +
         "https://github.com/develar/traveling-ruby/releases/download/v#{TRAVELING_RUBY_VERSION}/#{gem_name_and_version}-#{TRAVELING_RUBY_VERSION}-#{target}.tar.xz"
end