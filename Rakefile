require 'rubygems'
require 'rake'
require 'rake/testtask'
require 'rdoc/task'
require 'rake/packagetask'
require 'rubygems/package_task'
require File.join(File.dirname(__FILE__), 'lib', 'action_pack', 'version')

PKG_BUILD     = ENV['PKG_BUILD'] ? '.' + ENV['PKG_BUILD'] : ''
PKG_NAME      = 'actionpack'
PKG_VERSION   = ActionPack::VERSION::STRING + PKG_BUILD
PKG_FILE_NAME = "#{PKG_NAME}-#{PKG_VERSION}"

RELEASE_NAME  = "REL #{PKG_VERSION}"

RUBY_FORGE_PROJECT = "actionpack"
RUBY_FORGE_USER    = "webster132"

desc "Default Task"
task :default => [ :test ]

# Run the unit tests

desc "Run all unit tests"
task :test => [:test_action_pack, :test_active_record_integration]

Rake::TestTask.new(:test_action_pack) do |t|
  t.libs << "test"

  # make sure we include the tests in alphabetical order as on some systems
  # this will not happen automatically and the tests (as a whole) will error
  t.test_files = Dir.glob( "test/[cftv]*/**/*_test.rb" ).sort

  t.verbose = true
  #t.warning = true
end

desc 'ActiveRecord Integration Tests'
Rake::TestTask.new(:test_active_record_integration) do |t|
  t.libs << "test"
  t.test_files = Dir.glob("test/activerecord/*_test.rb")
  t.verbose = true
end


# Genereate the RDoc documentation

RDoc::Task.new { |rdoc|
  rdoc.rdoc_dir = 'doc'
  rdoc.title    = "Action Pack -- On rails from request to response"
  rdoc.options << '--line-numbers' << '--inline-source'
  rdoc.options << '--charset' << 'utf-8'
  rdoc.template = ENV['template'] ? "#{ENV['template']}.rb" : '../doc/template/horo'
  if ENV['DOC_FILES'] 
    rdoc.rdoc_files.include(ENV['DOC_FILES'].split(/,\s*/))
  else
    rdoc.rdoc_files.include('README', 'RUNNING_UNIT_TESTS', 'CHANGELOG')
    rdoc.rdoc_files.include(Dir['lib/**/*.rb'] -
                            Dir['lib/*/vendor/**/*.rb'])
    rdoc.rdoc_files.exclude('lib/actionpack.rb')
  end
}

# Create compressed packages
dist_dirs = [ "lib", "test" ]

spec = Gem::Specification.new do |s|
  s.platform = Gem::Platform::RUBY
  s.name = PKG_NAME
  s.version = PKG_VERSION
  s.summary = "Web-flow and rendering framework putting the VC in MVC."
  s.description = %q{Eases web-request routing, handling, and response as a half-way front, half-way page controller. Implemented with specific emphasis on enabling easy unit/integration testing that doesn't require a browser.} #'

  s.author = "David Heinemeier Hansson"
  s.email = "david@loudthinking.com"
  s.rubyforge_project = "actionpack"
  s.homepage = "http://www.rubyonrails.org"

  s.requirements << 'none'

  s.add_dependency('activesupport', '= 2.3.17' + PKG_BUILD)
  s.add_dependency('rack', '> 1.1.3')

  s.require_path = 'lib'

  s.files = [ "Rakefile", "install.rb", "README", "RUNNING_UNIT_TESTS", "CHANGELOG", "MIT-LICENSE" ]
  dist_dirs.each do |dir|
    s.files = s.files + Dir.glob( "#{dir}/**/*" ).delete_if { |item| item.include?( "\.svn" ) }
  end
end
  
Gem::PackageTask.new(spec) do |p|
  p.gem_spec = spec
  p.need_tar = true
  p.need_zip = true
end

task :lines do
  lines, codelines, total_lines, total_codelines = 0, 0, 0, 0

  for file_name in FileList["lib/**/*.rb"]
    next if file_name =~ /vendor/
    f = File.open(file_name)

    while line = f.gets
      lines += 1
      next if line =~ /^\s*$/
      next if line =~ /^\s*#/
      codelines += 1
    end
    puts "L: #{sprintf("%4d", lines)}, LOC #{sprintf("%4d", codelines)} | #{file_name}"
    
    total_lines     += lines
    total_codelines += codelines
    
    lines, codelines = 0, 0
  end

  puts "Total: Lines #{total_lines}, LOC #{total_codelines}"
end

# Publishing ------------------------------------------------------

task :update_scriptaculous do
  for js in %w( controls dragdrop effects )
    system("svn export --force http://dev.rubyonrails.org/svn/rails/spinoffs/scriptaculous/src/#{js}.js #{File.dirname(__FILE__)}/lib/action_view/helpers/javascripts/#{js}.js")
  end
end

desc "Updates actionpack to the latest version of the javascript spinoffs"
task :update_js => [ :update_scriptaculous ]

# Publishing ------------------------------------------------------

desc "Publish the API documentation"
task :pgem => [:package] do 
  require 'rake/contrib/sshpublisher'
  Rake::SshFilePublisher.new("gems.rubyonrails.org", "/u/sites/gems/gems", "pkg", "#{PKG_FILE_NAME}.gem").upload
  `ssh gems.rubyonrails.org '/u/sites/gems/gemupdate.sh'`
end

desc "Publish the API documentation"
task :pdoc => [:rdoc] do 
  require 'rake/contrib/sshpublisher'
  Rake::SshDirPublisher.new("wrath.rubyonrails.org", "public_html/ap", "doc").upload
end

desc "Publish the release files to RubyForge."
task :release => [ :package ] do
  require 'rubyforge'
  require 'rake/contrib/rubyforgepublisher'

  packages = %w( gem tgz zip ).collect{ |ext| "pkg/#{PKG_NAME}-#{PKG_VERSION}.#{ext}" }

  rubyforge = RubyForge.new
  rubyforge.login
  rubyforge.add_release(PKG_NAME, PKG_NAME, "REL #{PKG_VERSION}", *packages)
end
