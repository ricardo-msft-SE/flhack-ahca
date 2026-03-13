source "https://rubygems.org"

gem "jekyll", "~> 4.3"

group :jekyll_plugins do
  gem "jekyll-seo-tag", "~> 2.7"
  gem "jekyll-sitemap", "~> 1.4"
  gem "jekyll-remote-theme", "~> 0.4"
end

# Windows / JRuby timezone data
platforms :mingw, :x64_mingw, :mswin, :jruby do
  gem "tzinfo", ">= 1", "< 3"
  gem "tzinfo-data"
end

# Directory watcher performance boost on Windows
gem "wdm", "~> 0.1.1", platforms: %i[mingw x64_mingw mswin]

# Lock http_parser.rb on JRuby
gem "http_parser.rb", "~> 0.6.0", platforms: [:jruby]
