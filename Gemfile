source "https://rubygems.org"

# GitHub Pages 호환을 위해 jekyll 대신 github-pages 사용
gem "github-pages", group: :jekyll_plugins

# Just-the-docs 테마는 GitHub Pages에서 지원하는 방식으로 설정
gem "just-the-docs"

# Windows 및 특정 플랫폼 관련 설정 (필요 시)
platforms :mingw, :x64_mingw, :mswin, :jruby do
  gem "tzinfo", ">= 1", "< 3"
  gem "tzinfo-data"
end

# Performance-booster for watching directories on Windows
gem "wdm", "~> 0.1", :platforms => [:mingw, :x64_mingw, :mswin]

# Lock `http_parser.rb` gem to `v0.6.x` on JRuby builds since newer versions of the gem
# do not have a Java counterpart.
gem "http_parser.rb", "~> 0.6.0", :platforms => [:jruby]

