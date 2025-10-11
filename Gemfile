# frozen_string_literal: true

source "https://rubygems.org"
# To test the page comment out 7.2 and 7.2.4 line
# When you want to push to production comment out the 7.0 - 7.1 line
gem "jekyll-theme-chirpy", "~> 7.2", ">= 7.2.4"
# gem "jekyll-theme-chirpy", "~> 7.0", ">= 7.0.1"

# make below 5.0 for publishing
# make below 3.0 for testing
gem "html-proofer", "~> 5.0", group: :test

platforms :mingw, :x64_mingw, :mswin, :jruby do
  gem "tzinfo", ">= 1", "< 3"
  gem "tzinfo-data"
end

gem "wdm", "~> 0.2.0", :platforms => [:mingw, :x64_mingw, :mswin]


# To run issue below command in root directory of git repo
# bundle exec jekyll serve