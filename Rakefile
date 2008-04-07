# Copyright (C) 2008 Dag Odenhall <dag.odenhall@gmail.com>
# Licensed under the Academic Free License version 3.0

%w[rubygems yaml haml erb bluecloth redcloth rack].each do |lib|
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
  def link_style(stylesheet="*", media=:screen)
    Dir["styles/#{stylesheet}.sass"].map do |style|
      "<link href='stylesheets/#{shortfile(style)}.css' media='#{media}' rel='stylesheet' type='text/css' />"
    end.join("\n")
  end

  def link_to(page, text=nil)
    "<a href='#{page}.html'>#{text || page.to_s.capitalize}</a>"
  end
end

task :default => [:compile]

directory "public"
directory "public/stylesheets"

def load_view_variables(view, o)
  if File.exist?("configs/views.yml")
    @config_views ||= YAML.load_file("configs/views.yml") rescue {}
    @config_views.each {|key, value| @config_views[key.to_s] = value }
    @config_views[view].each do |key, value|
      o.instance_variable_set "@#{key}".to_sym, value
    end if @config_views.include?(view)
    o.instance_variable_set :@view, view
  end
end

rule ".html" => proc {|view| "views/#{shortfile(view)}.markdown" } do |t|
  File.open(t.name, "w+") do |f|
    scope = Object.new
    load_view_variables(shortfile(t.name), scope)
    include ERB::Util
    include Helpers
    f.write(Haml::Engine.new(File.read("layouts/default.haml"), :filename => "layouts/default.haml").to_html(scope) {
      BlueCloth.new(ERB.new(File.read(t.source)).result(scope.instance_eval { binding() })).to_html
    })
  end
end

rule ".html" => proc {|view| "views/#{shortfile(view)}.textile" } do |t|
  File.open(t.name, "w+") do |f|
    scope = Object.new
    load_view_variables(shortfile(t.name), scope)
    include ERB::Util
    include Helpers
    f.write(Haml::Engine.new(File.read("layouts/default.haml"), :filename => "layouts/default.haml").to_html(scope) {
      RedCloth.new(ERB.new(File.read(t.source)).result(scope.instance_eval { binding() })).to_html
    })
  end
end

rule ".html" => proc {|view| "views/#{shortfile(view)}.erb" } do |t|
  File.open(t.name, "w+") do |f|
    scope = Object.new
    load_view_variables(shortfile(t.name), scope)
    include ERB::Util
    include Helpers
    f.write(Haml::Engine.new(File.read("layouts/default.haml"), :filename => "layouts/default.haml").to_html(scope) {
      ERB.new(File.read(t.source)).result(scope.instance_eval { binding() })
    })
  end
end

rule ".html" => proc {|view| "views/#{shortfile(view)}.haml" } do |t|
  File.open(t.name, "w+") do |f|
    scope = Object.new
    load_view_variables(shortfile(t.name), scope)
    include ERB::Util
    include Helpers
    f.write(Haml::Engine.new(File.read("layouts/default.haml"), :filename => "layouts/default.haml").to_html(scope) {
      Haml::Engine.new(File.read(t.source), :filename => t.source).to_html(scope)
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

directory "configs"
directory "layouts"
directory "views"
directory "styles"

desc "Generate scaffold style, layout and view"
task :scaffold => ["configs", "styles", "layouts", "views"] do
  unless File.exist?("configs/views.yml")
    File.open("configs/views.yml", "w+") do |f|
      f.write(<<eof)
index:
  title: Ruby on Warp drive&#8212;Home
eof
    end
  end
  unless File.exist?("layouts/default.haml")
    File.open("layouts/default.haml", "w+") do |f|
      f.write(<<eof)
!!! XML
!!! Strict
%html{html_attrs}
  %head
    %meta{"http-equiv" => "Content-Type", "content" => "text/html, charset=utf-8"}
    %title= @title
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
