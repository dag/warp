# Copyright (C) 2008 Dag Odenhall <dag.odenhall@gmail.com>
# Licensed under the Academic Free License version 3.0

%w[rubygems yaml haml erb bluecloth redcloth rdoc/markup/simple_markup rdoc/markup/simple_markup/to_html rack].each do |lib|
  begin
    require lib
  rescue LoadError
  end
end

class File
  def self.shortname(filename)
    basename(filename[0..-extname(filename).length-1])
  end
end

PAGES = Dir["views/*"].map {|p| "public/#{File.shortname(p)}.html" }
STYLES = Dir["styles/*"].map {|s| "public/stylesheets/#{File.shortname(s)}.css" }

module Warp
  module Helpers
    def link_style(stylesheet="*", media=:screen)
      Dir["styles/#{stylesheet}.sass"].map do |style|
        "<link href='stylesheets/#{File.shortname(style)}.css' media='#{media}' rel='stylesheet' type='text/css' />"
      end.join("\n")
    end

    def link_to(page, text=nil)
      "<a href='#{page}.html'>#{text || page.to_s.capitalize}</a>"
    end
  end

  class Page
    include ERB::Util
    include Helpers

    def initialize(view)
      @view = view
      @page = File.shortname(view)
      if File.exist?("configs/pages.yml")
        @@config_pages ||= YAML.load_file("configs/pages.yml") rescue {}
        @@config_pages.each {|key, value| @@config_pages[key.to_s] = value }
        @@config_pages[@page].each do |key, value|
          instance_variable_set "@#{key}".to_sym, value
        end if @@config_pages.include?(@page)
      end
      @layout ||= "default"
    end

    def to_html
      render_layout { render_view }
    end

    private

    def render_layout
      layout_file = Dir["layouts/#@layout.*"].first
      case File.extname(layout_file)[1..-1].to_sym
      when :haml
        Haml::Engine.new(File.read(layout_file), :filename => layout_file).to_html(self) { yield }
      when :erb
        ERB.new(File.read(layout_file)).result(binding()) { yield }
      when :textile
        RedCloth.new(ERB.new(File.read(layout_file)).result(binding())).to_html { yield }
      when :markdown
        BlueCloth.new(ERB.new(File.read(layout_file)).result(binding())).to_html { yield }
      end
    end

    def render_view
      case File.extname(@view)[1..-1].to_sym
      when :markdown
        BlueCloth.new(ERB.new(File.read(@view)).result(binding())).to_html
      when :textile
        RedCloth.new(ERB.new(File.read(@view)).result(binding())).to_html
      when :haml
        Haml::Engine.new(File.read(@view), :filename => @view).to_html(self)
      when :erb
        ERB.new(File.read(@view)).result(binding())
      when :rdoc
        SM::SimpleMarkup.new.convert(ERB.new(File.read(@view)).result(binding()), SM::ToHtml.new)
      end
    end
  end
end

module Rack
  module Adapter
    class Warp
      def initialize
        @app = File.new("public")
      end

      def call(env)
        request = Request.new(env)
        request.path_info = "/index.html" if request.path_info == "/"
        @app.call(env)
      end
    end
  end
end

task :default => [:compile]

directory "public"
directory "public/stylesheets"

%w(markdown textile erb haml rdoc).each do |view_type|
  rule ".html" => proc {|view| "views/#{File.shortname(view)}.#{view_type}" } do |t|
    File.open(t.name, "w+") {|f| f.write(Warp::Page.new(t.source).to_html) }
  end
end

rule ".css" => proc {|task_name| "styles/#{File.shortname(task_name)}.sass" } do |t|
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
  unless File.exist?("configs/pages.yml")
    File.open("configs/pages.yml", "w+") do |f|
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
