require 'yaml'
require 'uri'

CONFIG = YAML.load_file("_config.yml")

def g(key)
  CONFIG['buildDocs'][ key ]
end

def update_wiki_submodule
  cd g('wiki_source') do
    pullCommand = 'git pull origin master'
    puts "Updating wiki submodule"
    output = `#{pullCommand}`

    if output.include? 'Already up-to-date'
      abort("No update necessary")
    end
  end
end

def clean_wiki_folders
  if File.exist?(g('wiki_dest'))
    puts "remove older wiki pages"

    Dir.glob("#{g('wiki_dest')}/*.md") do |wikiPage|
      puts "removing #{g('wiki_dest')}"
      rm_rf wikiPage
    end
  else
    puts "create the dest dir for wiki pages"
    FileUtils.mkdir(g('wiki_dest'))
  end
end

def copy_wiki_pages
    Dir.glob("#{g('wiki_source')}/[A-Za-z]*.md") do |wikiPage|

      wikiPageFileName = File.basename(wikiPage) 
      wikiPagePath     = File.join("#{g('wiki_dest')}", wikiPageFileName)
      wikiPageName    = wikiPageFileName.sub(/.[^.]+\z/,'')
      wikiPageTitle   = wikiPageName.gsub("-"," ")
      fileContent      = File.read(wikiPage)

      puts "generating #{wikiPagePath}"
      open(wikiPagePath, 'w') do |newWikiPage|
        newWikiPage.puts "# #{wikiPageTitle}"
        newWikiPage.puts ""
        newWikiPage.puts fileContent
      end

    end
end

def deploy
    puts "deploying"
    system "git add -A"
    message = "Site wiki update #{Time.now.utc}"
    puts "\n## Committing: #{message}"
    system "git commit -m \"#{message}\""
    puts "\n## Pushing website"
    system "git push #{g('deploy_remote')} #{g('deploy_branch')}"
    puts "\n## Github Pages deploy complete"
end

task :sync do |t|
    update_wiki_submodule
    Rake::Task[:wikibuild].execute
    if g('commit_and_push') == true
        deploy
    end
    puts "Wiki synchronisation successful!"
end

task :wikisub do |t|

  puts "adding wiki as submodule"
  wiki_repository = g('wiki_repository_url')
  command = 'git submodule add ' + wiki_repository + ' ' + g('wiki_source')
  command += ' && git submodule init'
  command += ' && git submodule update'
  puts 'command : ' + command

  output = `#{command}`

  if output.include? 'failed'
    abort("submodule add failed : verify you configuration and that you wiki is public")
  end

  puts "wiki submodule OK"
end


task :wikibuild do |t|
  puts 'rake:wikibuild'
  clean_wiki_folders
  copy_wiki_pages
end

task :deploy do |t|
    deploy
end
