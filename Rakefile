# Rakefile for RubyGems      -*- ruby -*-

require 'rubygems'
require 'rake/clean'
require 'rake/testtask'
require 'rake/packagetask'
require 'rake/gempackagetask'
require 'rake/rdoctask'
require 'rake/contrib/sshpublisher'
require 'net/ftp'

PKG_NAME = 'rubyzip'
PKG_VERSION = File.read('lib/zip/zip.rb').match(/\s+VERSION\s*=\s*'(.*)'/)[1]

PKG_FILES = FileList.new

PKG_FILES.add %w{ README NEWS TODO install.rb Rakefile }
PKG_FILES.add %w{ samples/*.rb }
PKG_FILES.add %w{ test/*.rb }
PKG_FILES.add %w{ test/data/* }
PKG_FILES.exclude "test/data/generated"
PKG_FILES.add %w{ lib/**/*.rb }

def clobberFromCvsIgnore(path)
  CLOBBER.add File.readlines(path+'/.cvsignore').map { 
    |f| File.join(path, f.chomp) 
  }
end

clobberFromCvsIgnore '.'
clobberFromCvsIgnore 'samples'
clobberFromCvsIgnore 'test'
clobberFromCvsIgnore 'test/data'

task :default => [:test]

desc "Run unit tests"
task :test do
  ruby %{-C test alltests.rb}
end

# Shortcuts for test targets
task :ut => [:test]

spec = Gem::Specification.new do |s|
  s.name = PKG_NAME
  s.version = PKG_VERSION
  s.author = "Thomas Sondergaard"
  s.email = "thomas(at)sondergaard.cc"
  s.homepage = "http://rubyzip.sourceforge.net/"
  s.platform = Gem::Platform::RUBY
  s.summary = "rubyzip is a ruby module for reading and writing zip files"
  s.files = PKG_FILES.to_a #Dir.glob("{samples,lib,test,docs}/**/*").delete_if {|item| item.include?("CVS") || item.include?("rdoc") || item =~ /~$/ }
  s.require_path = 'lib'
  s.autorequire = 'zip/zip'
end

Rake::GemPackageTask.new(spec) do |pkg|
  pkg.need_zip = true
  pkg.need_tar = true
end

Rake::RDocTask.new do |rd|
  rd.main = "README"
  rd.rdoc_files.add %W{ lib/zip/*.rb README NEWS TODO  }
  rd.options << "--title 'rubyzip documentation' --webcvs http://cvs.sourceforge.net/viewcvs.py/rubyzip/rubyzip/"
#  rd.options << "--all"
end

desc "Publish documentation"
task :pdoc => [:rdoc] do
  Rake::SshFreshDirPublisher.
       new("thomas@rubyzip.sourceforge.net", "rubyzip/htdocs", "html").upload
end

desc "Publish package"
task :ppackage => [:package] do
  Net::FTP.open("upload.sourceforge.net", 
                "ftp", 
                ENV['USER']+"@"+ENV['HOSTNAME']) {
    |ftpclient|
    ftpclient.chdir "incoming"
    Dir['pkg/*.{tgz,zip,gem}'].each {
      |e|
      ftpclient.putbinaryfile(e, File.basename(e))
    }
  }
end
