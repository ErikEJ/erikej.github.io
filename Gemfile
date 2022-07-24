source 'https://rubygems.org'

gem "minima", "~> 2.5.1"

gem 'github-pages', group: :jekyll_plugins

group :jekyll_plugins do
  gem "jekyll-feed", "~> 0.9"
end

install_if -> { RUBY_PLATFORM =~ %r!mingw|mswin|java! } do
  gem 'tzinfo', '~> 1.2'
  gem 'tzinfo-data'
end

gem 'wdm', '~> 0.1.0', :install_if => Gem.win_platform?
