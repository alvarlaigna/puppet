# -*- ruby -*- (Make emacs happy)
# Rakefile for Puppet

PKG = "puppet"
RPMHOST = "fedora1"
PKGHOST = "culain"

begin
    require 'rake/reductive'
rescue Exception
    $stderr.puts "You must have the Reductive build library in your RUBYLIB."
    exit(14)
end


begin
    require 'rubygems'
    require 'rake/gempackagetask'
rescue Exception
    nil
end

$rdoc = true
begin
    require 'rdoc/rdoc'
rescue => detail
    $rdoc = false
    puts "No rdoc: %s" % detail
end
require 'rake/clean'
require 'rake/testtask'

require 'facter'

if $rdoc
    require 'rake/rdoctask'
end
CLEAN.include('**/*.o')
CLOBBER.include('doc/*')

def announce(msg='')
    STDERR.puts msg
end

# Determine the current version

if `ruby -Ilib ./bin/puppet --version` =~ /\S+$/
    CURRENT_VERSION = $&
else
    CURRENT_VERSION = "0.0.0"
end

if ENV['REL']
  PKG_VERSION = ENV['REL']
else
  PKG_VERSION = CURRENT_VERSION
end

DOWNDIR = "/export/docroots/reductivelabs.com/htdocs/downloads"

if ENV['HOSTS']
    TESTHOSTS = ENV['HOSTS'].split(/\s+/)
else
    TESTHOSTS = %w{fedora1 rh3a culain openbsd1 centos1}
end

OS = Facter["operatingsystem"].value

#TESTHOSTS = %w{sol10b}

# The default task is run if rake is given no explicit arguments.

$sitedir = $:.find { |d| d =~ /site_ruby$/ }

unless $sitedir
    raise "Could not find site_ruby directory"
end

desc "Default Task"
task :default => :alltests

# Test Tasks ---------------------------------------------------------

task :u => :unittests
task :a => :alltests

task :alltests do
    sh %{cd test; ./test}
end

#Rake::TestTask.new(:alltests) do |t|
#    t.test_files = FileList['test/*/*.rb']
#    t.warning = true
#    t.verbose = false
#end

#Rake::TestTask.new(:unittests) do |t|
#    t.test_files = FileList['test/test']
#    t.warning = true
#    t.verbose = false
#end

# SVN Tasks ----------------------------------------------------------
# ... none.

# Install rake using the standard install.rb script.

desc "Install the application"
task :install do
    ruby "install.rb"
end

# Create a task to build the RDOC documentation tree.

#Rake::RDocTask.new("ri") { |rdoc|
#    #rdoc.rdoc_dir = 'html'
#    #rdoc.template = 'html'
#    rdoc.title    = "Puppet"
#    rdoc.options << '--ri' << '--line-numbers' << '--inline-source' << '--main' << 'README'
#    rdoc.rdoc_files.include('README', 'LICENSE', 'TODO', 'CHANGELOG')
#    rdoc.rdoc_files.include('lib/**/*.rb', 'doc/**/*.rdoc')
#}

if $rdoc
    Rake::RDocTask.new(:html) { |rdoc|
        rdoc.rdoc_dir = 'html'
        rdoc.template = 'html'
        rdoc.title    = "Puppet"
        rdoc.options << '--line-numbers' << '--inline-source' << '--main' << 'README'
        rdoc.rdoc_files.include('README', 'LICENSE', 'TODO', 'CHANGELOG')
        rdoc.rdoc_files.include('lib/**/*.rb')
        CLEAN.include("html")
    }
end

if $rdoc
    task :ri do |ri|
        files = ['README', 'LICENSE', 'TODO', 'CHANGELOG'] + Dir.glob('lib/**/*.rb')
        puts "files are \n%s" % files.join("\n")
        begin
            ri = RDoc::RDoc.new
            ri.document(["--ri-site"] + files)
        rescue RDoc::RDocError => detail
            puts "Failed to build docs: %s" % detail
            return nil
        rescue LoadError
            puts "Missing rdoc; cannot build documentation"
            return nil
        end
    end
end

# ====================================================================
# Create a task that will package the Rake software into distributable
# tar, zip and gem files.

PKG_FILES = FileList[
    'install.rb',
    '[A-Z]*',
    'lib/**/*.rb',
    'test/**/*.rb',
    'bin/**/*',
    'ext/**/*',
    'examples/**/*',
    'conf/**/*'
]
PKG_FILES.delete_if {|item| item.include?(".svn")}

if ! defined?(Gem)
    puts "Package Target requires RubyGEMs"
else
    spec = Gem::Specification.new { |s|

        #### Basic information.

        s.name = 'puppet'
        s.version = PKG_VERSION
        s.summary = "Puppet is a server configuration management tool."
        s.description = <<-EOF
Puppet is a declarative language for expressing system configuration,
a client and server for distributing it, and a library for realizing 
the configuration.
        EOF
        s.platform = Gem::Platform::RUBY

        #### Dependencies and requirements.

        # I'd love to explicitly list all of the libraries that I need,
        # but gems seem to only be able to handle dependencies on other
        # gems, which is, um, stupid.
        s.add_dependency('facter', '>= 1.1.0')
        #s.requirements << ""

        s.files = PKG_FILES.to_a

        #### Load-time details: library and application (you will need one or both).

        s.require_path = 'lib'                         # Use these for libraries.

        s.bindir = "bin"                               # Use these for applications.
        s.executables = ["puppet", "puppetd", "puppetmasterd", "puppetdoc",
                         "puppetca"]
        s.default_executable = "puppet"
        s.autorequire = 'puppet'

        #### Documentation and testing.

        s.has_rdoc = true
        #s.extra_rdoc_files = rd.rdoc_files.reject { |fn| fn =~ /\.rb$/ }.to_a
        s.rdoc_options <<
            '--title' <<  'Puppet - Configuration Management' <<
            '--main' << 'README' <<
            '--line-numbers'
        s.test_file = "test/test"

        #### Signing key and cert chain
        #s.signing_key = '/..../gem-private_key.pem'
        #s.cert_chain = ['gem-public_cert.pem']

        #### Author and project details.

        s.author = "Luke Kanies"
        s.email = "dev@reductivelabs.com"
        s.homepage = "http://reductivelabs.com/projects/puppet"
        s.rubyforge_project = "puppet"
    }

    Rake::GemPackageTask.new(spec) { |pkg|
        pkg.need_tar = true
    }
    CLEAN.include("pkg")
end

# Misc tasks =========================================================

#ARCHIVEDIR = '/tmp'

#task :archive => [:package] do
#  cp FileList["pkg/*.tgz", "pkg/*.zip", "pkg/*.gem"], ARCHIVEDIR
#end

# Define an optional publish target in an external file.  If the
# publish.rf file is not found, the publish targets won't be defined.

#load "publish.rf" if File.exist? "publish.rf"

# Support Tasks ------------------------------------------------------

def egrep(pattern)
    Dir['**/*.rb'].each do |fn|
        count = 0
        open(fn) do |f|
            while line = f.gets
        count += 1
        if line =~ pattern
            puts "#{fn}:#{count}:#{line}"
        end
            end
        end
    end
end

desc "Look for TODO and FIXME tags in the code"
task :todo do
    egrep "/#.*(FIXME|TODO|TBD)/"
end

#desc "Look for Debugging print lines"
#task :dbg do
#  egrep /\bDBG|\bbreakpoint\b/
#end

#desc "List all ruby files"
#task :rubyfiles do 
#  puts Dir['**/*.rb'].reject { |fn| fn =~ /^pkg/ }
#  puts Dir['**/bin/*'].reject { |fn| fn =~ /svn|(~$)|(\.rb$)/ }
#end

# --------------------------------------------------------------------
# Creating a release

desc "Make a new release"
task :release => [
        :prerelease,
        :clobber,
        :update_version,
        :tag, # tag everything before we make a bunch of extra dirs
        :html,
        :package,
        :fedorarpm,
        :copy
      ] do
  
    announce 
    announce "**************************************************************"
    announce "* Release #{PKG_VERSION} Complete."
    announce "* Packages ready to upload."
    announce "**************************************************************"
    announce 
end

# Validate that everything is ready to go for a release.
task :prerelease do
    announce 
    announce "**************************************************************"
    announce "* Making RubyGem Release #{PKG_VERSION}"
    announce "* (current version #{CURRENT_VERSION})"
    announce "**************************************************************"
    announce  

    # Is a release number supplied?
    unless ENV['REL']
        fail "Usage: rake release REL=x.y.z [REUSE=tag_suffix]"
    end

    # Is the release different than the current release.
    # (or is REUSE set?)
    if PKG_VERSION == CURRENT_VERSION && ! ENV['REUSE']
        fail "Current version is #{PKG_VERSION}, must specify REUSE=tag_suffix to reuse version"
    end

    # Are all source files checked in?
    if ENV['RELTEST']
        announce "Release Task Testing, skipping checked-in file test"
    else
        announce "Checking for unchecked-in files..."
        data = `svn -q update`
        unless data =~ /^$/
            fail "SVN update is not clean ... do you have unchecked-in files?"
        end
        announce "No outstanding checkins found ... OK"
    end
end

task :update_version => [:prerelease] do
    if PKG_VERSION == CURRENT_VERSION
        announce "No version change ... skipping version update"
    else
        announce "Updating Puppet version to #{PKG_VERSION}"
        open("lib/puppet.rb") do |rakein|
            open("lib/puppet.rb.new", "w") do |rakeout|
                rakein.each do |line|
                    if line =~ /^(\s*)PUPPETVERSION\s*=\s*/
                        rakeout.puts "#{$1}PUPPETVERSION = '#{PKG_VERSION}'"
                    else
                        rakeout.puts line
                    end
                end
            end
        end
        mv "lib/puppet.rb.new", "lib/puppet.rb"

        open("conf/redhat/puppet.spec") do |rakein|
            open("conf/redhat/puppet.spec.new", "w") do |rakeout|
                rakein.each do |line|
                    if line =~ /^Version:\s*/
                        rakeout.puts "Version: #{PKG_VERSION}"
                    elsif line =~ /^Release:\s*/
                      rakeout.puts "Release: 1%{?dist}"
                    else
                        rakeout.puts line
                    end
                end
            end
        end
        mv "conf/redhat/puppet.spec.new", "conf/redhat/puppet.spec"

        if ENV['RELTEST']
            announce "Release Task Testing, skipping commiting of new version"
        else
            sh %{svn commit -m "Updated to version #{PKG_VERSION}" lib/puppet.rb conf/redhat/puppet.spec}
        end
    end
end

desc "Copy the newly created package into the downloads directory"
task :copy => [:package, :html, :fedorarpm] do
    puts Dir.getwd
    sh %{cp pkg/puppet-#{PKG_VERSION}.gem #{DOWNDIR}/gems}
    sh %{generate_yaml_index.rb -d #{DOWNDIR}}
    sh %{cp pkg/puppet-#{PKG_VERSION}.tgz #{DOWNDIR}/puppet}
    sh %{ln -sf puppet-#{PKG_VERSION}.tgz #{DOWNDIR}/puppet/puppet-latest.tgz}
    sh %{cp -r html #{DOWNDIR}/puppet/apidocs}
    sh %{rsync -av /home/luke/rpm/. #{DOWNDIR}/rpm}
end

desc "Tag all the SVN files with the latest release number (REL=x.y.z)"
task :tag => [:prerelease] do
    reltag = "REL_#{PKG_VERSION.gsub(/\./, '_')}"
    reltag << ENV['REUSE'].gsub(/\./, '_') if ENV['REUSE']
    announce "Tagging SVN copy with [#{reltag}]"
    if ENV['RELTEST']
        announce "Release Task Testing, skipping SVN tagging"
    else
        sh %{svn copy ../trunk/ ../tags/#{reltag}}
        sh %{cd ../tags; svn ci -m "Adding release tag #{reltag}"}
    end
end

desc "Test Puppet on each test host"
task :hosttest do
    out = ""
    TESTHOSTS.each { |host|
        puts "testing %s" % host
        cwd = Dir.getwd
        system("ssh #{host} 'cd puppet/test; sudo ./test' 2>&1 >/tmp/#{host}test.out")

        if $? != 0
            puts "%s failed; output is in %s" % [host, file]
        end
        #sh %{ssh #{host} 'cd #{cwd}/test; sudo ./test' 2>&1} 
    }

    #IO.popen("mail -s 'Puppet Test Results' luke@madstop.com") do |m|
    #    m.puts out
    #end
end

desc "Create an RPM"
task :rpm do
    tarball = File.join(Dir.getwd, "pkg", "puppet-#{PKG_VERSION}.tgz")

    sourcedir = `rpm --define 'name puppet' --define 'version #{PKG_VERSION}' --eval '%_sourcedir'`.chomp
    specdir = `rpm --define 'name puppet' --define 'version #{PKG_VERSION}' --eval '%_specdir'`.chomp
    basedir = File.dirname(sourcedir)

    if ! FileTest::exist?(sourcedir)
        FileUtils.mkdir_p(sourcedir)
    end
    FileUtils.mkdir_p(basedir)

    target = "#{sourcedir}/#{File::basename(tarball)}"

    sh %{cp %s %s} % [tarball, target]
    sh %{cp conf/redhat/puppet.spec %s/puppet.spec} % basedir

    Dir.chdir(basedir) do
        sh %{rpmbuild -ba puppet.spec}
    end

    sh %{mv %s/puppet.spec %s} % [basedir, specdir]
end

desc "Create an rpm on a system that can actually do so"
task :fedorarpm => [:package] do
    sh %{ssh fedora1 'cd puppet; rake rpm'}
end

def epmlist(match, prefix = "/usr")
    dest = "../epmtmp/#{OS}-#{match}"

    list = %x{mkepmlist --prefix #{prefix} ../puppet-#{PKG_VERSION}}.gsub(/luke/, "0")

    list = list.split(/\n/).find_all do |line|
        line =~ /#{prefix}\/#{match}/
    end.join("\n")

    File.open(dest, "w") { |f| f.puts list }

    return dest
end

directory "pkg/epm"
directory "pkg/epmtmp"

desc "Create packages using EPM"
task :epmpkg => ["pkg/epm", "pkg/epmtmp", :package] do
    $epmdir = "pkg/epm"
    $epmtmpdir = "pkg/epmtmp"

    Dir.chdir($epmdir) do
        type = nil

        binfile = epmlist("bin", "/usr")
        libfile = epmlist("lib", $sitedir)

        listfile = "../epmtmp/#{OS}.list"
        sh %{cat ../../conf/epm.list #{binfile} #{libfile} > #{listfile}}
        sh %{epm -f native puppet #{listfile}}
    end
end

desc "Make all of the appropriate packages"
task :allepmpkgs => [:package] do
    %w{freebsd1 culain}.each do |host|
        sh %{ssh #{host} 'cd puppet; rake epmpkg'}
    end
end

# $Id$
