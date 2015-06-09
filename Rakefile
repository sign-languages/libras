require 'rake/clean'
require 'yaml'
require 'tilt'
require 'liquid'
require 'json'

LANGUAGE_CONF = YAML.load_file('config/language.yaml')
SLANG = YAML.load_file('config/language.yaml')['lang_dir']
TARGET = 'target'

directory TARGET

CLEAN.include("#{TARGET}")

desc "Generate asciidoc files"
task :adoc

desc "Generate html files"
task :html

tags = {}
tags_dir = "#{TARGET}/tags"

desc "Create tags files"
task :tags

def template_file (title)
  YAML.load_file('config/templates.yaml')[title]
end

json_language_file = "#{TARGET}/language.json"
file json_language_file
json_language = {
  title: LANGUAGE_CONF['title'],
  url:LANGUAGE_CONF['url'],
  version:LANGUAGE_CONF['version'],
  date: Time.now.utc,
  keys:{}
  }

Rake::FileList["#{SLANG}/**/*.yaml"].each do |file|
  key = file.ext("")[(SLANG.size+1)..-1]  # libras/c/casa.yaml -> c/casa

  adoc_parent_dir = "#{TARGET}/#{key}"  # target/c/casa
  directory adoc_parent_dir => TARGET

  adoc_file = "#{adoc_parent_dir}/#{File.basename(file)}".ext(".adoc") # target/c/casa/casa.adoc
  conf = YAML.load_file(file)
  json_language[:keys][key]=conf
  file adoc_file => [file, adoc_parent_dir,template_file('adoc')] do |t|
    template = Tilt.new(template_file('adoc'))
    rendered = template.render(conf, key:key)
    File.open(t.name, 'w') { |f| f.write(rendered) }
  end
  task :adoc => adoc_file

  html_file = "#{adoc_parent_dir}/index.html"
  file html_file => [file, adoc_parent_dir, adoc_file] do
    system "asciidoctor -a linkcss #{adoc_file} -o #{html_file}"
  end
  task :html => [html_file]

   # save tags
   conf['tags'].each do |tag|
     if not tags[tag]
       tags[tag] = []
     end
     tags[tag] << key

     tag_dir =  "#{tags_dir}/#{tag}"
     directory tag_dir
     tag_adoc = "#{tag_dir}/index.adoc"
     file tag_adoc => [tag_dir, html_file]
     task :tags => tag_adoc
     file json_language_file => [tag_adoc]
   end

   file json_language_file => [html_file]
end

tags.each do |tag, keys|
  tag_dir =  "#{tags_dir}/#{tag}"
  tag_adoc = "#{tag_dir}/index.adoc"
  file tag_adoc do |t|
    template = Tilt.new(template_file('tag'))
    rendered = template.render(tag: tag, keys: keys)
    File.open(t.name, 'w') { |f| f.write(rendered) }
  end
  tag_html = tag_adoc.ext('.html')
  file tag_html => tag_adoc do |t|
    system "asciidoctor -a linkcss #{tag_adoc} -o #{tag_html}"
  end
  task :tags => tag_html
end

file template_file('tags')

tags_adoc =  "#{tags_dir}/index.adoc"
file tags_adoc => [tags_dir, template_file('tags')] do |t|
  template = Tilt.new(template_file('tags'))
  rendered = template.render(tags: tags)
  File.open(t.name, 'w') { |f| f.write(rendered) }
end

tags_html = "#{tags_dir}/index.html"
file tags_html => [tags_adoc] do |t|
  system "asciidoctor -a linkcss #{tags_adoc} -o #{tags_html}"
end
task :html => tags_html

task :adoc => tags_adoc
task :tags => tags_html


file json_language_file do |t|
  File.open(t.name, 'w') { |f| f.write(JSON.generate(json_language)) }
end

desc 'Generate the json file of language'
task :json => json_language_file


task :compile

task :playlist do
  require 'google/api_client'
  client = Google::APIClient.new
  urlshortener = client.discovered_api('urlshortener')
end
