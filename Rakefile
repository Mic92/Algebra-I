require 'rubygems'
require 'rake/clean'
require "mime/types"

require 'github_api'

# Dieses Ruby-Script lädt halb-automatisch die neuen PDF-Dokumente hoch.
# Dafür wird das Passwort meines Github-Accounts benötigt.
# Es wird eine Standartinstallation von ruby benötigt.
# und das github_api gem
# Installation
# $ apt-get install ruby && gem install github_api
# Benutzung
# $ rake upload
# Desweiteren kann es PDF-Dokumente erstellen
# $ rake pdf

def config
  # in this Rakefile we can allow that
  $config ||= Hash.new
end

## Latex Settings
# main TeX file without extension
config['main'] = 'script'
# TeX command to invoke: xelatex, pdflatex, etc
config['latex'] = 'pdflatex'
# path to inkscape (to convert svg -> pdf)
config['inkscape'] = 'inkscape'
# path to rubber (a convienent latex wrapper [optional])
config['rubber'] = 'rubber'

# picture path
config['picture-path'] = 'bilder'

## Upload Settings
# Repo name
config['github-repo'] = 'Algebra-I'
# Owner of the github repo
config['repo-owner']  = 'Mic92'
# File which should be upload to github
config['upload-file'] = 'script.pdf'
# path to curl
config['curl'] = 'curl'

task :default => [ :pdf ]

SVG_PICS = FileList[File.join(config['picture-path'], '*.svg')]
PDF_PICS = SVG_PICS.ext('pdf')

[ 'aux', 'bbl', 'blg', 'pdf', 'toc', 'nav', 'log', 'out' ].each do |res|
    CLEAN.include(Dir.glob("*.#{res}"))
end

desc 'Build pdf with latex'
task :pdf => [ :info, :clean, :pictures ] do
  if config['has-rubber'] then
      exec 'rubber', '--pdf', "#{config['main']}.tex"
  else
      exec 'latex', '-halt-on-error', "#{config['main']}.tex"
  end
end

rule '.pdf' => ['.svg'] do |t|
    exec 'inkscape', "--file='#{t.source}'",
        '--export-area-drawing',
        '--without-gui',
        "--export-pdf='#{t.name}'"
end

file PDF_PICS => SVG_PICS
task :pictures => PDF_PICS

desc 'Print configuration'
task :info do
  puts "Configuration: #{config.inspect}"

  has_latex = exec 'latex', '--version' rescue false
  raise 'LaTeX is not installed' unless has_latex
  config['has-rubber'] = exec 'rubber', '--version' rescue false
end

desc 'Upload pdf to github'
task :upload => [:pdf] do
    user = `git config github.user`.chomp
    file = config['upload-file']
    #token = `git config github.token`.chomp
    print "password[#{user}]: "
    system 'stty -echo'
    password = STDIN.gets.chomp
    system 'stty echo'
    puts


    github = Github.new login: user, password: password

    downloads = github.repos.downloads.list config['repo-owner'], config['github-repo']
    downloads.each do |download|
        if download.name == File.basename(file) then
            puts 'Delete old download...'
            github.repos.downloads.delete config['repo-owner'],
                config['github-repo'], download.id
        end
    end

    content_type = MIME::Types.type_for(file)[0].to_s

    puts "Create download"
    resource = github.repos.downloads.create config['repo-owner'],
        config['github-repo'],
        :name => File.basename(file),
        :size => File.size(file),
        :description => 'Algebra Script',
        :content_type => content_type

    puts 'Upload new download...'
    uploader = Github::S3Uploader.new resource, file
    uploader.send
    puts '...finished!'
end


def exec(command, *arguments)
  raise "no configuration found for #{command}" unless config[command]
  args = arguments.flatten.map { |arg| "'#{arg}'" }.join(' ')
  sh "#{config[command]} #{args}"
end
