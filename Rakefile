# Copyright (C) 2008 Dag Odenhall <dag.odenhall@gmail.com>
# Licensed under the Academic Free License version 3.0

%w[rubygems haml bluecloth redcloth].each do |lib|
  begin
    require lib
  rescue LoadError
  end
end

def shortfile(filename)
  File.basename(filename[0..-File.extname(filename).length-1])
end

PAGES = Dir["views/*"].map {|p| "public/#{shortfile(p)}.html" }
STYLES = Dir["styles/*"].map {|s| "public/stylesheets/#{shortfile(s)}.css" }

module Helpers
  def link_style(stylesheet="*")
    Dir["styles/#{stylesheet}.sass"].map do |style|
      "<link href='stylesheets/#{shortfile(style)}.css' media='screen' rel='stylesheet' type='text/css' />"
    end.join("\n")
  end

  def link_to(page, text=nil)
    "<a href='#{page}.html'>#{text || page.to_s.capitalize}</a>"
  end
end

task :default => [:compile]

directory "public"
directory "public/stylesheets"

rule ".html" => proc {|view| "views/#{shortfile(view)}.markdown" } do |t|
  File.open(t.name, "w+") do |f|
    include Helpers
    f.write(Haml::Engine.new(File.read("layouts/default.haml")).to_html {
      BlueCloth.new(File.read(t.source)).to_html
    })
  end
end

rule ".html" => proc {|view| "views/#{shortfile(view)}.textile" } do |t|
  File.open(t.name, "w+") do |f|
    include Helpers
    f.write(Haml::Engine.new(File.read("layouts/default.haml")).to_html {
      RedCloth.new(File.read(t.source)).to_html
    })
  end
end

rule ".css" => proc {|task_name| "styles/#{shortfile(task_name)}.sass" } do |t|
  File.open(t.name, "w+") do |f|
    f.write(Sass::Engine.new(File.read(t.source)).render)
  end
end

desc "Toss generated content"
task :clean do
  (PAGES + STYLES).each {|f| rm f rescue nil }
  rmdir "public/stylesheets" rescue nil
  rmdir "public" rescue nil
end

desc "Compile pages and stylesheets"
task :compile => ["public", "public/stylesheets"] + PAGES + STYLES

desc "Recompile pages and stylesheets"
task :recompile => [:clean, :compile]

directory "layouts"
directory "views"
directory "styles"

file "layouts/default.haml" => ["layouts"] do |t|
  File.open(t.name, "w+") do |f|
    f.write(<<eof)
!!! XML
!!! Strict
%html{html_attrs}
  %head
    %title Ruby on Warp drive!
    = link_style
  %body
    #container
      #banner
        %h1 Ruby on Warp drive!
      #nav
        %ul
          %li= link_to :index, "Home"
      #content= yield
eof
  end
end

file "views/index.markdown" => ["views"] do |t|
  File.open(t.name, "w+") do |f|
    f.write(<<eof)
Features
--------

* Cool
* Sexy
* Fast
* Secure
* Simple
eof
  end
end

file "styles/global.sass" => ["styles"] do |t|
  File.open(t.name, "w+") do |f|
    f.write(<<eof)
!blue = #4183C4

*
  font-family: helvetica,arial,clean,sans-serif

ul
  list-style-type: square

a
  color= !blue
  text-decoration: none
eof
  end
end

desc "Generate scaffold style, layout and view"
task :scaffold => ["styles/global.sass", "layouts/default.haml", "views/index.markdown"]
