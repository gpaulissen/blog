source "https://rubygems.org"

# To suppress this error:
#
# Invalid theme folder: _sass
gem 'jekyll', '3.7.4'

jekyll_env = ENV['JEKYLL_ENV'] || 'development'

if jekyll_env == 'development'
then
  gem "minimal-mistakes-jekyll"
  gem 'html-proofer'
  gem 'rake'
end
gem "github-pages", group: :jekyll_plugins

# If you have any plugins, put them here!
group :jekyll_plugins do
  gem "jekyll-gist"
  gem "jekyll-sitemap"
  gem "jekyll-paginate"
  gem "jekyll-include-cache"
  gem "jekyll-remote-theme"
end
