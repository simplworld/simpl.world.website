#!/usr/bin/env ruby

require 'html-proofer'

task :test do
  sh 'bundle exec jekyll build'
  options = {
    :assume_extension => true,
    :checks_to_ignore => [
        'ImageCheck'
    ],
    :file_ignore => [
        /404.html/
    ],
    :only_4xx => true,
    :url_ignore => []
  }
  HTMLProofer.check_directory("./_site", options).run
end

task :default => 'test'
