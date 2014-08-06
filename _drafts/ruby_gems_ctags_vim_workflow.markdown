---
layout: post
title:  "ctags with git, ruby, gems, vim, Tim Pope, Drew Neil, sausage-filler samovar, dragons and unicorns"
date:   2014-08-03 09:23
author: toraritte
categories: code ruby git vim ctags gem
---
<section><h3>Caveat</h3>
  Please note that none of the solutions and workflows are devised by me. Some of the text is copied verbatim and it is altered only in places where it conflicted with my setup.
</section>
<nav><h3>Overview</h3>
  <ol>
    <li><a href='#git'>git</a></li>
    <li><a href='#bibliography'>Bibliography</a></li>
  </ol>
</nav>
<section><h3 id='git'>git</h3>
  <p>Tim Pope created a git template and several hooks so that <q cite="http://tbaggery.com/2011/08/08/effortless-ctags-with-git.html">any new repositories you create or clone will be immediately indexed with Ctags and set up to re-index every time you check out, commit, merge, or rebase. Basically, you’ll never have to manually run Ctags on a Git repository again.</q><a href="#b01">[1]</a></p>

  Set up a default set of hooks that Git will use as a template when creating or cloning a repository (requires Git 1.7.1 or newer):

  {% highlight console %}
  git config --global init.templatedir '~/dotfiles/.git_template'
  mkdir -p ~/.git_template/hooks
  {% endhighlight %}

  Be aware of the 'dotfiles' directory above! It is a personal setup!

  Now onto the first hook, which isn’t actually a hook at all, but rather a script the other hooks will call. Place in <code>.git_template/hooks/ctags</code> and mark as executable:

  {% highlight bash %}
  #!/bin/sh
  set -e
  PATH="/usr/local/bin:$PATH"
  trap "rm -f .git/tags.$$" EXIT
  ctags --tag-relative -Rf.git/tags.$$ --exclude=.git --languages=-javascript,sql
  mv .git/tags.$$ .git/tags
  {% endhighlight %}

  This will be the result at the end:

  {% highlight console %}
  $ ll .git_template/hooks/
  total 28
  drwxr-xr-x 2 toraritte toraritte 4096 Apr  8 20:38 ./
  drwxr-xr-x 3 toraritte toraritte 4096 Apr  8 20:09 ../
  -rwxr-xr-x 1 toraritte toraritte  182 Apr  8 20:33 ctags*
  -rwxr-xr-x 1 toraritte toraritte   45 Apr  8 20:37 post-checkout*
  -rwxr-xr-x 1 toraritte toraritte   45 Apr  8 20:37 post-commit*
  -rwxr-xr-x 1 toraritte toraritte   45 Apr  8 20:37 post-merge*
  -rwxr-xr-x 1 toraritte toraritte   68 Apr  8 20:38 post-rewrite*
  {% endhighlight %}

  Making this a separate script makes it easy to invoke <code>.git/hooks/ctags</code> for a one-off re-index (or <code>git config --global alias.ctags '!.git/hooks/ctags'</code>, then <code>git ctags</code>).
  I stick the tags file in .git because if fugitive.vim is installed, Vim will be configured to look for it there automatically, regardless of your current working directory. Plus, you don’t need to worry about adding it to .gitignore.

  Here come the hooks. Mark all four of them executable and place them
  in .git_template/hooks. Use this same content for the first three:
  post-commit, 
  post-merge, and 
  post-checkout (actually my post-checkout hook includes hookup as well).

  {% highlight bash %}
  #!/bin/sh
  .git/hooks/ctags >/dev/null 2>&1 &
  {% endhighlight %}

  I’ve forked it into the background so that my Git workflow remains as 
  latency-free as possible.

  One more hook that oftentimes gets overlooked: post-rewrite. This is 
  fired after git commit --amend and git rebase, but the former is 
  already covered by post-commit. Here’s mine:

  {% highlight bash %}
  #!/bin/sh
  case "$1" in
  rebase) exec .git/hooks/post-merge ;;
  esac
  {% endhighlight %}

  Once you get this all set up, you can use git init in existing 
  repositories to copy these hooks in. (and git ctags alias will work)
</section>
<section><h3 id='bibliography'>Bibliography</h3>
  <ol>
    <div itemscope itemtype="http://schema.org/BlogPosting">
      <li id="b01">
        <span itemprop="author" itemscope itemtype="http://schema.org/Person">
          <span itemprop="name">Pope, Tim</span>
        </span>
        <cite itemprop="name">Effortless Ctags with Git</cite>
        <time datetime="2011-08-08" itemprop="datePublished" value='2011-08-08'>2011-08-08</time>
        <a href="http://tbaggery.com/2011/08/08/effortless-ctags-with-git.html" itemprop="url">http://tbaggery.com/2011/08/08/effortless-ctags-with-git.html</a>
      </li>
    </div>
  </ol>
</section>
