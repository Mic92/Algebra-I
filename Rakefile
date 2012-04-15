require 'rubygems'
require 'rake'
require "mime/types"

require 'github_api'

repo_owner = "Mic92"
repo = 'Algebra-I'
file = 'script.pdf'

task :upload do
    user = `git config github.user`.chomp
    #token = `git config github.token`.chomp
    print "password[#{user}]: "
    system("stty -echo")
    password = STDIN.gets.chomp
    system("stty echo")
    puts

    github = Github.new :basic_auth => "#{user}:#{password}"

    downloads = github.repos.downloads(repo_owner, repo)
    downloads.each do |download|
        if download.name == File.basename(file) then
            puts "delete old download..."
            github.repos.delete_download repo_owner, repo, download.id
        end
    end

    content_type = MIME::Types.type_for(file)[0].to_s

    resource = github.repos.create_download repo_owner, repo,
        :name => File.basename(file),
        :size => File.size(file),
        :description => "Algebra Script",
        :content_type => content_type

    puts "upload new download..."
    system("curl",
           "-F", "key=#{resource.path}",
           "-F", "acl=#{resource.acl}",
           "-F", "success_action_status=201",
           "-F", "Filename=#{resource.name}",
           "-F", "AWSAccessKeyId=#{resource.accesskeyid}",
           "-F", "Policy=#{resource.policy}",
           "-F", "Signature=#{resource.signature}",
           "-F", "Content-Type=#{resource.mime_type[0]['Content-Type']}",
           "-F", "file=@#{file}",
                       resource.s3_url)
    puts "\n"
    puts "...finished!"
end
