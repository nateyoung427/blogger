## Java Never Sleep

A blogger focusing on Java only. It's all about Java. Java never sleep, me not, me sleep:)

## Notes
* Initial `about`, `categories` and `tags`
```bash
hexo new page about
hexo new page tags
hexo new page categories
```
After that you may customize `index.md` under each directory, for `tags` and `categories`, please add `type: "tags"` or `type: "cateogries"`
* Create a `CNAME` file under `source` folder and add purchased domain name there, for example: www.javaneversleep.com
* To avoid user permission issue in Ubuntu, use `nvm` to install nodejs and then `npm install hexo-cli -g`
* For disqus, the `shortname` is not that of user name in profile, but the Sites you created in Admin panel, fail to do this will prevent disqus comments area to display correctly

## Dependencies
* local search: `npm install hexo-generator-searchdb --save`
* symbols count: `npm install hexo-symbols-count-time --save`
* sitemap generator: `npm install hexo-generator-sitemap --save`
* hexo deploy: `npm install hexo-deployer-git --save`
* related posts: `npm install hexo-related-popular-posts --save`
* needmoreshare:
    ```bash
    cd themes\next
    git clone https://github.com/theme-next/theme-next-needmoreshare2 source/lib/needsharebutton
    ```