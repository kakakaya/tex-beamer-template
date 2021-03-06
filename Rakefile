# coding: utf-8
# MICS Report - Rakefile
require 'yaml'

SRC = "tex"
TMP = "tmp"
OUTPUT = "output"
CONFIG = "config"
LIB = "lib"
CODE = "src"
IMAGES = "images"

# init / reset project files
task :init do
  `mkdir #{SRC}`
  `mkdir #{TMP}`
  `mkdir #{OUTPUT}`
  `mkdir #{CONFIG}`
  `mkdir #{CODE}`
  `mkdir #{IMAGES}`

  `cp #{LIB}/title.yml.sample #{CONFIG}/title.yml`

  `touch #{CONFIG}/tex_files`
  `touch #{TMP}/title.tex`
  `touch #{TMP}/body.tex`
  `touch #{TMP}/footer.tex`
  `echo "\\end{document}" > #{TMP}/footer.tex`

  `git remote remove origin`
end

task :reset do
  `rm -rf #{SRC}`
  `rm -rf #{TMP}`
  `rm -rf #{OUTPUT}`
  `rm -rf #{CONFIG}`
  `rm -rf #{CODE}`
  `rm -rf #{IMAGES}`
end

task :clean do
  # 一時ファイルのみ削除する、参照が壊れた時に使う
  `rm -rf #{TMP}`
  `mkdir #{TMP}`
  `echo "\\end{document}" > #{TMP}/footer.tex`
end

# create new .tex file
task :create do
  if ENV['name'].nil?
    puts "ERROR: NO NAME"
    puts "execute `rake create name=NAME`"
    exit
  end

  `touch #{SRC}/#{ENV['name']}.tex`
  `echo #{ENV['name']}.tex >> #{CONFIG}/tex_files`
end

task :make_title do
  title = YAML.load_file("#{CONFIG}/title.yml")
  `cat #{LIB}/title.tex > #{TMP}/title.tex`
  `cat #{LIB}/slide_header.tex > #{TMP}/slide_header.tex`
  `cat #{LIB}/handout_header.tex > #{TMP}/handout_header.tex`

  # 各種変数
  `echo '\\\\title{#{title['title']}}' >> #{TMP}/title.tex`
  `echo '\\\\author{#{title['name']}}' >> #{TMP}/title.tex`
  `echo '\\\\date{#{title['date']}}' >> #{TMP}/title.tex`
  `echo '\\\\begin{document}' >> #{TMP}/title.tex`

  # 表紙
  `echo '\\\\begin{frame}[plain]\\\\frametitle{}' >> #{TMP}/title.tex`
  `echo '\\\\titlepage' >> #{TMP}/title.tex`
  `echo '\\\\end{frame}' >> #{TMP}/title.tex`

  # 目次
  `echo '\\\\begin{frame}\\\\frametitle{Contents}' >> #{TMP}/title.tex`
  `echo '\\\\tableofcontents' >> #{TMP}/title.tex`
  `echo '\\\\end{frame}' >> #{TMP}/title.tex`
end

task :make_body do
  cmd = ["cat"]
  open "#{CONFIG}/tex_files" do |file|
    while l = file.gets
      cmd << "#{SRC}/#{l.chomp}"
    end
  end

  `#{cmd.join(" ")} > #{TMP}/body.tex` if cmd.size > 1
  `cp #{CODE}/* #{TMP}/`

  `cp #{IMAGES}/*.png #{TMP}/`
  `cp #{IMAGES}/*.jpg #{TMP}/`
  `cp #{IMAGES}/*.eps #{TMP}/`
  `cd #{TMP} && extractbb *.png`
end

task :concat do
end

task :make_slide do
  `cat #{TMP}/slide_header.tex #{TMP}/title.tex #{TMP}/body.tex #{TMP}/footer.tex > #{TMP}/slide.tex`

  `cd #{TMP} && uplatex slide.tex && uplatex slide.tex`
  `cd #{TMP} && dvipdfmx slide.dvi`
  `mv #{TMP}/slide.pdf #{OUTPUT}/slide.pdf`
end

task :make_handout do
  `cat #{TMP}/handout_header.tex #{TMP}/title.tex #{TMP}/body.tex #{TMP}/footer.tex > #{TMP}/handout.tex`

  `cd #{TMP} && uplatex handout.tex && uplatex handout.tex`
  `cd #{TMP} && dvipdfmx handout.dvi`
  `mv #{TMP}/handout.pdf #{OUTPUT}/handout.pdf`
end

task :make_pdf_debug do
  # デバッグ用のメッセージを出力しながらPDFを作成する
  `cat #{TMP}/slide_header.tex #{TMP}/title.tex #{TMP}/body.tex #{TMP}/footer.tex > #{TMP}/debug.tex`

  sh "cd #{TMP} && uplatex debug.tex && uplatex debug.tex"
  sh "cd #{TMP} && dvipdfmx debug.dvi"
  `mv #{TMP}/debug.pdf #{OUTPUT}/debug.pdf`
end

task :copy_host do
  # ~/mnt/host に output/*.pdfをコピーする
  `cd #{OUTPUT} && ls -1 | xargs -n 1 -I % sh -c "echo % && cp % \"$HOME/mnt/host/$(python3 -c 'import os; print("_".join(os.getcwd().split("/")[-3:-1]))')-%\""`
end

task :compile => [:make_title, :make_body, :concat, :make_slide, :make_handout]

task :default => :compile
