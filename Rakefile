# coding: utf-8

# Copyright 2017 Andreas Krüger, dj3ei@famsik.de
# 
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
# 
#     http://www.apache.org/licenses/LICENSE-2.0
# 
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

require 'net/http'
require 'nokogiri'

@host = 'www.dspguide.com'
@user_agent = 'DJ3EI\'s homebrew ebook-maker'
@client = Net::HTTP.new(@host, 80)

def grab_html_body(path, reccount = 0, client = @client)
  raise "Too much redirection: #{path}" unless reccount <= 5
  response = client.request_get(path, {'user-agent' => @user_agent})
  if response.is_a?(Net::HTTPSuccess)
    if /^text\/html(;.*)?$/i =~ response['content-type']
      return response.body
    else
      raise "Don't know how to deal with content-type \"#{response['content-type']}\""
    end
  elsif response.is_a?(Net::HTTPFound)
    newpath = response['location']
    $stderr.print "INFO: Redirection from #{path} to #{newpath}\n"
    raise "Insane: Too many redirects." unless reccount < 5
    if /^http:\/\/#{@host}(\/.+)$/ =~ newpath
      grab_html_body($1, reccount + 1, @client)
    else
      raise "Do not follow redirect to #{newpath}"
    end
  else
    raise "No success HTTP GET #{path} on #{@client.inspect}: #{response.inspect}"
  end
end

desc "do it all"
task :default => [ :write_start_document, :grab_chapters, :make_book ]

desc 'Make the book itself.'
task :make_book do
  system('ebook-convert', 'start.html', 'dspguide.epub', '--max-levels=3') \
    or raise "Could not ebook-convert: $?"
end

desc 'Write the start / table of content document.'
task :write_start_document do
  File.open('start.html','w') do |out|
    out <<'
<html>
  <head>
    <meta http-equiv="Content-Type" content="text/html; charset=utf-u" />
    <title>The Scientist and Engineer\'s Guide to Digital Signal Processing</title>
  </head>
  <body>
    <ul>
'
    (1 .. 34).each do |ch|
      out << "      <li><a href=\"ch#{ch}.htm\">Chapter #{ch}</a></li>\n"
    end
    out << '    </ul>
  </body>
</html>
'
  end
end

def cleanup_html(doc, level = "")
  # This could be streamlined.
  doc.xpath('//link[@rel="stylesheet"]').each do |css|
    css.remove
  end
  doc.xpath('//div[@id="header"]').each do |div|
    div.remove
  end
  doc.xpath('//div[@id="footer"]').each do |div|
    div.remove
  end
  doc.xpath('//div[@id="columnLeft"]').each do |div|
    div.remove
  end
  doc.xpath('//div[@id="adbox"]').each do |div|
    div.remove
  end
  doc.xpath('//div[@class="breadcrumbs"]').each do |div|
    div.remove
  end
  doc.xpath('//script').each do |script|
    script.remove
  end
  doc.xpath('//img[@src]').each do |img|
    img_path = img['src']
    if /^\/(graphics\/.+)$/ =~ img_path
      img_file = $1
      img_rel = "#{level}#{img_file}"
      img['src'] = img_rel
      img_response = @client.request_get(img_path, {'user-agent' => @user_agent})
      if img_response.is_a?(Net::HTTPSuccess)
        File.open(img_file, 'w') do |img_out|
          img_out.write(img_response.body)
        end
        img['style'] = 'width:95%;'
      else
        $stderr.puts "WARN: Could not retrieve #{img_path}, omitting."
        img.remove
      end
    else
      raise "ERROR: #{img.to_xhtml} not intended to be fetched."
    end
  end
end

desc 'Grab the individual .htm files and the graphics contained and adjust them.'
task :grab_chapters do
  File.directory?("graphics") or Dir.mkdir("graphics")
  (1 .. 34).each do |i|
    file = "ch#{i}.htm"
    File.directory?("ch#{i}") or Dir.mkdir("ch#{i}")
    path = "/#{file}"
    $stderr.puts "INFO: Grabbing #{path}"
    response = @client.request_get(path, {'user-agent' => @user_agent})
    if response.is_a?(Net::HTTPSuccess)
      if /^text\/html(;.*)?$/i =~ response['content-type']
        doc = Nokogiri::HTML(response.body)
        cleanup_html(doc)
        doc.xpath('//ul/li/a[@href]').each do |subcharlink|
          subchar = subcharlink['href']
          if /^ch#{i}\/.+.htm$/ =~ subchar
            sub_path = "/#{subchar}"
            sub_path_content = grab_html_body(sub_path)
            sub_doc = Nokogiri::HTML(sub_path_content)
            cleanup_html(sub_doc, '../')
            File.open(subchar, "w") do |sub_out|
              sub_out.write(sub_doc.to_xhtml)
            end
          else
            raise "ERROR: Programm can't parse subchar #{subchar}"
          end
        end
        File.open(file,"w") do |out|
          out.write(doc.to_xhtml)
        end
      else
        raise "Don't know how to deal with content-type \"#{response['content-type']}\""
      end
    else
      raise "ERROR: Unsuccessful response: #{response}"
    end
  end
end

desc '(Re-)generate the .gitignore file.'
task :make_gitignore do
  File.open('.gitignore', 'w') do |gi|
    gi << "dspguide.epub\nstart.html\ngraphics\n"
    (1 .. 34).each do |i|
      gi << "ch#{i}\nch#{i}.htm\n"
    end
  end
end

# Run
#    rake grab_all_pdfs
# to download the PDF files.
# Even if you don't have Nokogiri installed, this still works,
# after you remove the line "require 'nokogiri'" from the top of the file.
desc 'Early experiment: Grab all PDFs. Doesn\'t need Nokogiri, see source.'
task :grab_all_pdfs do |t|
  (1 .. 34).each do |i|
    file = "CH#{i}.PDF"
    path = "/#{file}"
    $stderr.puts "INFO: Grabbing #{path}"
    response = @client.request_get(path, {'user-agent' => @user_agent})
    if response.is_a?(Net::HTTPSuccess)
      if /^application\/pdf(;.*)?$/i =~ response['content-type']
        File.open(file,"w") do |out|
          out.write(response.body)
        end
      else
        raise "Don't know how to deal with content-type \"#{response['content-type']}\""
      end
    else
      raise "ERROR: Unsuccessful response: #{response}"
    end
  end
end
