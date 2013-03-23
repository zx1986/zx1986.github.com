guard 'livereload' do
  watch(%r{public/.+\.(css|js|html)$})
end

guard 'shell' do
  watch(%r{source/_posts/.+\.markdown$}) {`compass compile --css-dir source/stylesheets && jekyll`}
end
