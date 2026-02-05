# Mufradat Site

Hugo site served at https://mufradat.com

## Quick Edit

cd /var/www/mufradat
nano content/posts/azan-garmin-widget.md   # edit post
hugo --destination /var/www/html            # rebuild
git add -A && git commit -m "message" && git push

## Add New Post

cd /var/www/mufradat
hugo new content posts/my-new-post.md
nano content/posts/my-new-post.md          # edit, set draft: false
hugo --destination /var/www/html
git add -A && git commit -m "Add new post" && git push

## Add Images

Put images in static/images/ then reference in markdown as:
![Alt text](/images/myimage.jpg)

## File Structure

- content/posts/    - Blog posts
- content/about.md  - About page  
- content/impressum.md - Legal notice
- static/images/    - Images
- hugo.toml         - Site config

## Repo

https://github.com/ricksy/mufradat
