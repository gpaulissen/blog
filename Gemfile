source "https://rubygems.org"

require 'json'
require 'open-uri'
versions = JSON.parse(open('https://pages.github.com/versions.json').read)

gem 'tzinfo', '~> 1.2'
gem 'tzinfo-data', platforms: [:mingw, :mswin, :x64_mingw, :jruby]

# To suppress this error:
#
# Invalid theme folder: _sass
# gem 'jekyll', '3.7.4'
gem 'jekyll', versions['jekyll']

jekyll_env = ENV['JEKYLL_ENV'] || 'development'

if jekyll_env == 'development'
then
  gem "minimal-mistakes-jekyll"
  gem 'html-proofer'
  gem 'rake'
end
# gem "github-pages"
gem 'github-pages', versions['github-pages'], group: :jekyll_plugins

# If you have any plugins, put them here!
group :jekyll_plugins do
  gem "jekyll-gist", versions['jekyll-gist']
  gem "jekyll-sitemap"
  gem "jekyll-paginate", versions['jekyll-paginate']
  gem "jekyll-include-cache"
  gem "jekyll-remote-theme", versions['jekyll-remote-theme']
end

gem "addressable", ">= 2.8.0"
gem "nokogiri", ">= 1.12.5"
