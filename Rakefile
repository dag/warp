# Copyright (C) 2008 Dag Odenhall <dag.odenhall@gmail.com>
# Licensed under the Academic Free License version 3.0

%w[rubygems haml bluecloth redcloth rack].each do |lib|
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
    f.write(Haml::Engine.new(File.read("layouts/default.haml"), :filename => "layouts/default.haml").to_html {
      BlueCloth.new(File.read(t.source)).to_html
    })
  end
end

rule ".html" => proc {|view| "views/#{shortfile(view)}.textile" } do |t|
  File.open(t.name, "w+") do |f|
    include Helpers
    f.write(Haml::Engine.new(File.read("layouts/default.haml"), :filename => "layouts/default.haml").to_html {
      RedCloth.new(File.read(t.source)).to_html
    })
  end
end

rule ".css" => proc {|task_name| "styles/#{shortfile(task_name)}.sass" } do |t|
  File.open(t.name, "w+") do |f|
    f.write(Sass::Engine.new(File.read(t.source), :filename => t.source, :load_paths => ["styles"]).render)
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

desc "Generate scaffold style, layout and view"
task :scaffold => ["styles", "layouts", "views"] do
  unless File.exist?("layouts/default.haml")
    File.open("layouts/default.haml", "w+") do |f|
      f.write(<<eof)
!!! XML
!!! Strict
%html{html_attrs}
  %head
    %meta{"http-equiv" => "Content-Type", "content" => "text/html, charset=utf-8"}
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
  unless File.exist?("views/index.markdown")
    File.open("views/index.markdown", "w+") do |f|
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
  unless File.exist?("styles/global.sass")
    File.open("styles/global.sass", "w+") do |f|
      f.write(<<eof)
!blue = #4183C4

*
  :font-family helvetica,arial,clean,sans-serif

ul
  :list-style-type square

a
  :color = !blue
  :text-decoration none
eof
    end
  end
end

module Rack
  module Adapter
    class Warp
      def call(env)
        request = Rack::Request.new(env)
        if request.path_info.include? ".."
          return [403, {"Content-Type" => "text/plain"}, "Forbidden\n"]
        end

        if request.path_info == "/"
          @path = "public/index.html"
        else
          @path = ::File.join("public", Utils.unescape(request.path_info))
        end

        if ::File.file?(@path) && ::File.readable?(@path)
          [200, {
            "Last-Modified" => ::File.mtime(@path).rfc822,
            "Content-Type" => Rack::File::MIME_TYPES[::File.extname(@path)[1..-1]] || "text/plain",
            "Content-Length" => ::File.size(@path).to_s
          }, self]
        else
          return [404, {"Content-Type" => "text/plain"}, "File not found: #{request.path_info}\n"]
        end
      end

      def each
        ::File.open(@path, "rb") do |f|
          while part = f.read(8192)
            yield part
          end
        end
      end
    end
  end
end

desc "Serve content with Rack"
task :serve do
  handler = ENV["HANDLER"] || "Mongrel"
  port = ENV["PORT"] || 5000
  app = Rack::Adapter::Warp.new 
  begin
    Rack::Handler.const_get(handler).run(app, :Port => port)
  rescue LoadError
    Rack::Handler::WEBrick.run(app, :Port => port)
  rescue Interrupt
  end
end
