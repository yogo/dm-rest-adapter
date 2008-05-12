module DataMapper
  # Set this to the version of dm-core that you are building against/for
  VERSION = "0.9.0"

  # Set this to the version of dm-more you plan to release
  MORE_VERSION = "0.9.0"
end

require 'pathname'
require 'rake/clean'
require 'rake/gempackagetask'
require 'rake/contrib/rubyforgepublisher'
require 'spec/rake/spectask'
require 'rake/rdoctask'
require 'fileutils'
include FileUtils

DIR = Pathname(__FILE__).dirname.expand_path.to_s

gem_paths = %w[
  merb_datamapper
  dm-aggregates
  dm-ar-finders
  dm-cli
  dm-is-tree
  dm-migrations
  dm-serializer
  dm-timestamps
  dm-types
  dm-validations
  adapters/dm-couchdb-adapter
]
gems = gem_paths.map { |p| File.basename(p) }

PROJECT = "dm-more"

dm_more_spec = Gem::Specification.new do |s|
  s.platform = Gem::Platform::RUBY
  s.name = PROJECT
  s.summary = "An Object/Relational Mapper for Ruby"
  s.description = "Faster, Better, Simpler."
  s.version = DataMapper::MORE_VERSION

  s.authors = "Sam Smoot"
  s.email = "ssmoot@gmail.com"
  s.rubyforge_project = PROJECT
  s.homepage = "http://datamapper.org"

  s.files = %w[ MIT-LICENSE README Rakefile TODO lib/dm-more.rb ]
  s.add_dependency("dm-core", ">= #{DataMapper::VERSION}")
  gems.each do |gem|
    s.add_dependency gem, [">= #{DataMapper::VERSION}"]
  end
end

Rake::GemPackageTask.new(dm_more_spec) do |p|
  p.gem_spec = dm_more_spec
  p.need_tar = true
  p.need_zip = true
end

CLEAN.include ["**/.*.sw?", "pkg", "lib/*.bundle", "*.gem", "doc/rdoc", ".config", "coverage", "cache", "lib/merb-more.rb"]

windows = (PLATFORM =~ /win32|cygwin/) rescue nil

SUDO = windows ? "" : "sudo"

desc "Install it all"
task :install => [:install_gems, :package] do
  sh %{#{SUDO} gem install --local pkg/dm-more-#{DataMapper::MORE_VERSION}.gem  --no-update-sources}
#  sh %{#{SUDO} gem install --local pkg/dm-#{DataMapper::MORE_VERSION}.gem --no-update-sources}
end

desc "Build the dm-more gems"
task :build_gems do
  gem_paths.each do |dir|
    Dir.chdir(dir){ sh "rake gem" }
  end
end

desc "Install the dm-more gems"
task :install_gems => :build_gems do
  gem_paths.each do |dir|
    Dir.chdir(dir){ sh "#{SUDO} rake install" }
  end
end

desc "Uninstall the dm-more gems"
task :uninstall_gems do
  gems.each do |sub_gem|
    sh %{#{SUDO} gem uninstall #{sub_gem}}
  end
end

task :package => ["lib/dm-more.rb"]
desc "Create dm-more.rb"
task "lib/dm-more.rb" do
  mkdir_p "lib"
  File.open("lib/dm-more.rb","w+") do |file|
    file.puts "### AUTOMATICALLY GENERATED.  DO NOT EDIT."
    gems.each do |gem|
      next if gem == "dm-gen"
      file.puts "require '#{gem}'"
    end
  end
end

desc "Bundle up all the dm-more gems"
task :bundle => [:package, :build_gems] do
  mkdir_p "bundle"
#  cp "pkg/dm-#{DataMapper::MORE_VERSION}.gem", "bundle"
  cp "pkg/dm-more-#{DataMapper::MORE_VERSION}.gem", "bundle"
  gem_paths.each do |gem|
    File.open("#{gem}/Rakefile") do |rakefile|
      rakefile.read.detect {|l| l =~ /^VERSION\s*=\s*"(.*)"$/ }
      sh %{cp #{gem}/pkg/#{File.basename(gem)}-#{$1}.gem bundle/}
    end
  end
end

namespace :ci do

  gems.each do |gem_name|
    task gem_name do
      ENV['gem_name'] = gem_name

      Rake::Task["ci:run_all"].invoke
    end
  end

  task :run_all => [:spec, :install, :doc, :publish]

  task :spec_all => :define_tasks do
    gems.each do |gem_name|
      Rake::Task["#{gem_name}:spec"].invoke
    end
  end

  task :spec => :define_tasks do
    Rake::Task["#{ENV['gem_name']}:spec"].invoke
  end

  task :doc => :define_tasks do
    Rake::Task["#{ENV['gem_name']}:doc"].invoke
  end

  task :install do
    sh %{cd #{ENV['gem_name']} && rake install}
  end

  task :publish do
    out = ENV['CC_BUILD_ARTIFACTS'] || "out"
    mkdir_p out unless File.directory? out if out

    mv "rdoc", "#{out}/rdoc" if out
    mv "coverage", "#{out}/coverage_report" if out && File.exists?("coverage")
    mv "rspec_report.html", "#{out}/rspec_report.html" if out && File.exists?("rspec_report.html")
  end

  task :define_tasks do
    gem_names = [(ENV['gem_name'] || gems)].flatten
    gem_names.each do |gem_name|
      Spec::Rake::SpecTask.new("#{gem_name}:spec") do |t|
        puts "#{gem_name}:spec"
        t.spec_opts = ["--format", "specdoc", "--format", "html:rspec_report.html", "--diff"]
        t.spec_files = Pathname.glob(ENV['FILES'] || DIR + "/#{gem_name}/spec/**/*_spec.rb")
        unless ENV['NO_RCOV']
          t.rcov = true
          t.rcov_opts << '--exclude' << "spec,gems,#{(gems - [gem_name]).join(',')}"
          t.rcov_opts << '--text-summary'
          t.rcov_opts << '--sort' << 'coverage' << '--sort-reverse'
          t.rcov_opts << '--only-uncovered'
        end
      end

      Rake::RDocTask.new("#{gem_name}:doc") do |t|
        t.rdoc_dir = 'rdoc'
        t.title    = gem_name
        t.options  = ['--line-numbers', '--inline-source', '--all']
        t.rdoc_files.include("#{gem_name}/lib/**/*.rb", "#{gem_name}/ext/**/*.c")
      end
    end
  end
end
