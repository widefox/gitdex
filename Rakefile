require "bundler"
require "app/search"
require "app/repos"
Bundler.require(:default)

desc "Load all of the repositories into IndexTank, expects INDEXTANK_API_URL"
task :load_repos do

  # clean the repos dir
  system "rm -rf repos/*"

  api_base = IndexTank::Client.new ENV["INDEXTANK_API_URL"]
  repos_index = api_base.indexes "repos"
  
  available_repos.each do |repo_name|
    # clone the repo
    cmd = "git clone git://github.com/#{repo_name}.git repos/#{repo_name}"
    system cmd
    
    repo = Grit::Repo.new("repos/#{repo_name}")
    
    # iterate through at most 1,000,000 commits and store info about each
    repo.commits("master", 1000000).each do |commit|
      puts "Processing #{repo_name} / #{commit.sha}"

      begin
        # group info for the commit
        repo_info = {
          :repo => repo_name,
          :author => commit.author.name,
          :committer_name => commit.committer.name,
          :date => commit.date.to_i,
          :message => commit.message,
          :diff_files => commit.diffs.map { |d| d.b_path }.join(" "),
          :diff_patch => commit.diffs.map { |d| d.diff }.join(" ")
        }
      
        # store log info on IndexTank
        repos_index.document(commit.sha).add(repo_info)
      rescue
        puts "failed to store: #{commit.sha}"
      end
    end
  end
  
end


require "rake/testtask"
desc "Run unit tests"
Rake::TestTask.new("test") { |t|
  t.pattern = 'tests/*_test.rb'
  t.verbose = true
  t.warning = false
}
