require 'html-proofer'

task :default => :test

task :test do
  sh "bundle exec jekyll build --config _config.yml,_config_development.yml"
  options = { :assume_extension => true }
  HTMLProofer.check_directory("./_site", options).run
end

task :serve do
  sh "bundle exec jekyll serve --config _config.yml,_config_development.yml"
end
