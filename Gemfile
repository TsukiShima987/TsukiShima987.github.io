source "https://rubygems.org"

# 本地与 GitHub Actions 使用同一套 Jekyll 版本，
# 这样可以自由使用插件，不受 GitHub 内置构建器（Jekyll 3.9 白名单）的限制。
gem "jekyll", "~> 4.4.1"

# 官方 minima 主题。它自带 jekyll-feed 与 jekyll-seo-tag 依赖。
gem "minima", "~> 2.5"

group :jekyll_plugins do
  gem "jekyll-feed", "~> 0.17"
  gem "jekyll-seo-tag", "~> 2.8"
end

# Ruby 3.0 起 webrick 不再包含在标准库中，`jekyll serve` 需要它。
gem "webrick", "~> 1.8"

# Windows / JRuby 环境下时区数据缺失时启用。
platforms :mingw, :x64_mingw, :mswin, :jruby do
  gem "tzinfo", ">= 1", "< 3"
  gem "tzinfo-data"
end
