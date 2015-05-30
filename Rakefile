#require 'reveal-ck/rake_task'
require 'html/pipeline/abbr'
require 'dothtml/dot_task'

# RevealCK::GenerateTask.new do |t|
#   t.config.filters = %w(
#     HTML::Pipeline::AutoAbbrFilter
#     HTML::Pipeline::AbbrEmojiFilter
#     HTML::Pipeline::MentionFilter
#     HTML::Pipeline::AutolinkFilter
#   )
# end

Dothtml::DotTask.new do |t|
#  t.d3js      = "d3.v3.js"
#  t.style     = "style.css"
end
task :slides do
  puts "reveal-ck generate"
  `bundle exec reveal-ck generate`
end

task :default => :slides
task :html do
  Dir.glob("*.dot").each do |f|
    Rake::Task[f.sub(/\.dot$/, '.html')].invoke
  end
end

# RevealCK::ServeTask.new do |t|
#   t.config.filters = %w(
#     HTML::Pipeline::AutoAbbrFilter
#     HTML::Pipeline::AbbrEmojiFilter
#     HTML::Pipeline::MentionFilter
#     HTML::Pipeline::AutolinkFilter
#   )
# end
