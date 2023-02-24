# Starter kit for [Alembic](https://alembic.darn.es/)

This is a very simple starting point if you wish to use Alembic [as a Jekyll theme gem](https://alembic.darn.es/#as-a-jekyll-theme) or as a [GitHub Pages remote theme](https://github.com/daviddarnes/alembic-kit/tree/remote-theme) (see `remote-theme` branch).

[![Deploy to Netlify](https://www.netlify.com/img/deploy/button.svg)](https://app.netlify.com/start/deploy?repository=https://github.com/daviddarnes/alembic-kit)

or

**[Download the GitHub Pages kit](https://github.com/daviddarnes/alembic-kit/archive/remote-theme.zip)**

### How To Config: 
1. Create a github repo, and the repo name should be the same with the github usename;
2. Download the Github pages kit via the link above;
3. Add *gem "alembic-jekyll-theme"* to your **Gemfile** to add the theme as a dependancy；
4. Run the command *bundle install* in the root of project to install the theme and its dependancies；
5. Add *theme: alembic-jekyll-theme* to your **_config.yml** file to set the site theme；
6. Run *bundle exec jekyll serve* to build and serve your site
7. Done! Use the configuration documentation and the example _config.yml file to set things like the navigation, contact form and social sharing buttons


### Common Problems:
1. Solve the problem that gem source is too slow to download from abroad, use the chinese domestic mirror, after trying, *http://mirrors.aliyun.com/rubygems/ * is the fastest.
```
gem sources 　　--查看当前使用的源地址
gem sources -r https://rubygems.org/ 　　 --删除默认的源地址
gem sources -a http://mirrors.aliyun.com/rubygems/ 　　 --添加阿里云镜像地址
gem sources -u 　　 --更新源的缓存
```
