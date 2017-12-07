source "https://rubygems.org"
ruby RUBY_VERSION

# Which Jekyll version should we use?
#
# We want to keep up with what's on Github, see here for the latest:
# https://pages.github.com/versions/

gem "jekyll", "3.5.2"
gem "rake"
gem "github-pages", group: :jekyll_plugins
gem "rouge"
gem "kramdown"

# If you have any plugins, put them here!
group :jekyll_plugins do
  gem "jekyll-feed"
  gem "jekyll-redirect-from"
  gem "jekyll-seo-tag"
  gem "jekyll-sitemap"
end

group :development, :test do
  gem "html-proofer"
end
