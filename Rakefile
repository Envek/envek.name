
desc "Publish website"
task :publish do
  system('bash -c "jekyll build && rsync -az --delete _site/ envek@envek.name:/var/www/envek.name && echo Done."')
end
