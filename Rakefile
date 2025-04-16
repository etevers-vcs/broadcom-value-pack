namespace :book do

  # Variables referenced for build
  version_string = `git describe --tags --abbrev=0`.chomp
  if version_string.empty?
    version_string = '0'
  else
    versions = version_string.split('.')
    version_string = versions[0] + '.' + versions[1] + '.' + versions[2].to_i.next.to_s
  end
  date_string = Time.now.strftime('%Y-%m-%d')
  params = "--attribute revnumber='#{version_string}' --attribute revdate='#{date_string}'"
  header_hash = `git rev-parse --short HEAD`.strip

  # Check contributors list
  # This checks commit hash stored in the header of list against current HEAD
  def check_contrib
    if File.exist?('contributors.txt')
      current_head_hash = `git rev-parse --short HEAD`.strip
      header = `head -n 1 contributors.txt`.strip
      # Match regex, then coerce resulting array to string by join
      header_hash = header.scan(/[a-f0-9]{7,}/).join

      if header_hash == current_head_hash
        puts "Hash on header of contributors list (#{header_hash}) matches the current HEAD (#{current_head_hash})"
      else
        puts "Hash on header of contributors list (#{header_hash}) does not match the current HEAD (#{current_head_hash}), refreshing"
        sh "rm contributors.txt"
        # Reenable and invoke task again
        Rake::Task['contributors.txt'].reenable
        Rake::Task['contributors.txt'].invoke
      end
    end
  end

  desc 'build basic book formats'
  task :build => [:build_html, :build_epub, :build_fb2, :build_pdf] do
    begin
        # Run check
        Rake::Task['book:check'].invoke

        # Rescue to ignore checking errors
        rescue => e
        puts e.message
        puts 'Error when checking books (ignored)'
    end
  end

  desc 'build basic book formats (for ci)'
  task :ci => [:build_html, :build_epub, :build_fb2, :build_pdf] do
      # Run check, but don't ignore any errors
      Rake::Task['book:check'].invoke
  end

  desc 'generate contributors list'
  file 'contributors.txt' do
      puts 'Generating contributors list'
      sh "echo 'Contributors as of #{header_hash}:\n' > contributors.txt"
      sh "git shortlog -s HEAD | grep -v -E '(Straub|Chacon|dependabot)' | cut -f 2- | sort | column -c 120 >> contributors.txt"
  end

  desc 'build HTML format'
  task :build_html => 'contributors.txt' do
      check_contrib()

      puts 'Converting to HTML...'
      sh "bundle exec asciidoctor #{params} -a data-uri evcs-guide-architecture.asc"
      puts ' -- HTML output at evcs-guide-architecture.html'

  end

  desc 'build Epub format'
  task :build_epub => 'contributors.txt' do
      check_contrib()

      puts 'Converting to EPub...'
      sh "bundle exec asciidoctor-epub3 #{params} evcs-guide-architecture.asc"
      puts ' -- Epub output at evcs-guide-architecture.epub'

  end

  desc 'build FB2 format'
  task :build_fb2 => 'contributors.txt' do
      check_contrib()

      puts 'Converting to FB2...'
      sh "bundle exec asciidoctor-fb2 #{params} evcs-guide-architecture.asc"
      puts ' -- FB2 output at evcs-guide-architecture.fb2.zip'

  end

  desc 'build PDF format'
  task :build_pdf => 'contributors.txt' do
      check_contrib()
      # https://docs.asciidoctor.org/pdf-converter/latest/theme/cjk/
      puts 'Converting to PDF... (this one takes a while)'
      sh "bundle exec asciidoctor-pdf #{params} -a scripts=cjk -a pdf-theme=./themes/pdf/pdf-korean.yml -a pdf-fontsdir=./themes/pdf evcs-guide-architecture.asc 2>/dev/null"
      puts ' -- PDF output at evcs-guide-architecture.pdf'
  end

  desc 'Check generated books'
  task :check => [:build_html, :build_epub] do
      puts 'Checking generated books'

      sh "htmlproofer evcs-guide-architecture.html"
      sh "epubcheck evcs-guide-architecture.epub"
  end

  desc 'Clean all generated files'
  task :clean do
    begin
        puts 'Removing generated files'

        FileList['contributors.txt', 'evcs-guide-architecture.html', 'progit-kf8.epub', 'evcs-guide-architecture.epub', 'evcs-guide-architecture.fb2.zip', 'evcs-guide-architecture.pdf'].each do |file|
            rm file

            # Rescue if file not found
            rescue Errno::ENOENT => e
              begin
                  puts e.message
                  puts 'Error removing files (ignored)'
              end
        end
    end
  end

end

task :default => "book:build"
