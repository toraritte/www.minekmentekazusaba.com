to do:

1. add a rake task to extract TODOs from files in repo
   (a simple grep would suffice but I wanted to dig into rake anyway)

2. now the all blogpost will get loaded on the site root:
   - pagination

   - ugly code in <code>_layout/post.html</code> so that
     its markup could be reused in <code>index.html</code>
     any better solution? at least it's DRY

   - *corrolary of the previous point*:
     if many posts are iterated in <code>index.html</code>
     then how will it affect page load?
     (2 Liquid <code>include</code> with each post)
