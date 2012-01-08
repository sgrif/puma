require "hoe"
require "rake/extensiontask"
require "rake/javaextensiontask"

IS_JRUBY = defined?(RUBY_ENGINE) ? RUBY_ENGINE == "jruby" : false

HOE = Hoe.spec "puma" do
  self.rubyforge_name = 'puma'
  self.readme_file    = "README.md"

  developer 'Evan Phoenix', 'evan@phx.io'

  spec_extras[:extensions]  = ["ext/puma_http11/extconf.rb"]
  spec_extras[:executables] = ['puma', 'pumactl']

  dependency "rack", "~> 1.2"

  extra_dev_deps << ["rake-compiler", "~> 0.8.0"]
end

# hoe/test and rake-compiler don't seem to play well together, so disable
# hoe/test's .gemtest touch file thingy for now
HOE.spec.files -= [".gemtest"]

# puma.gemspec

file "#{HOE.spec.name}.gemspec" => ['Rakefile'] do |t|
  puts "Generating #{t.name}"
  File.open(t.name, 'wb') { |f| f.write HOE.spec.to_ruby }
end

desc "Generate or update the standalone gemspec file for the project"
task :gemspec => ["#{HOE.spec.name}.gemspec"]

# generate extension code using Ragel (C and Java)
desc "Generate extension code (C and Java) using Ragel"
task :ragel

file 'ext/puma_http11/http11_parser.c' => ['ext/puma_http11/http11_parser.rl'] do |t|
  begin
    sh "ragel #{t.prerequisites.last} -C -G2 -I ext/puma_http11 -o #{t.name}"
  rescue
    fail "Could not build wrapper using Ragel (it failed or not installed?)"
  end
end
task :ragel => ['ext/puma_http11/http11_parser.c']

file 'ext/puma_http11/org/jruby/puma/Http11Parser.java' => ['ext/puma_http11/http11_parser.java.rl'] do |t|
  begin
    sh "ragel #{t.prerequisites.last} -J -G2 -I ext/puma_http11 -o #{t.name}"
  rescue
    fail "Could not build wrapper using Ragel (it failed or not installed?)"
  end
end
task :ragel => ['ext/puma_http11/org/jruby/puma/Http11Parser.java']

# compile extensions using rake-compiler
# C (MRI, Rubinius)
Rake::ExtensionTask.new("puma_http11", HOE.spec) do |ext|
  # place extension inside namespace
  ext.lib_dir = "lib/puma"

  ext.cross_compile = true
  ext.cross_platform = ['i386-mswin32-60', 'i386-mingw32']

  CLEAN.include "lib/puma/{1.8,1.9}"
end

# Java (JRuby)
Rake::JavaExtensionTask.new("puma_http11", HOE.spec) do |ext|
  ext.lib_dir = "lib/puma"
end

# tests require extension be compiled, but depend on the platform
if IS_JRUBY
  task :test => [:java]
else
  task :test => [:compile]
end
