<div style="display:flex; gap:2rem; align-items:flex-start;">

  <div style="flex:3; min-width:0;" markdown="1">

![Banner](/assets/images/new_banner.png)

Welcome to the blog! I'm Shawn, a former manager and blue collar worker now employed in the world of offensive security. I'm hoping to share some of my journey and what I've learned along the way. I'm currently focused on web app pentesting and application security but constantly learning other areas of pentesting as well.

---

### Connect with me
<a href="https://linktr.ee/b1gb33f"
   target="_blank"
   rel="noopener noreferrer"
   style="padding:6px 12px; border:1px solid #ff2bd3; border-radius:4px; text-decoration:none; color:#ff2bd3; cursor:pointer;">
  Linktree
</a>

---

# Most Recent Post

{% assign most_recent_post = site.posts.first %}

## [{{ most_recent_post.title }}]({{ most_recent_post.url | relative_url }})
*{{ most_recent_post.date | date: "%B %d, %Y"}}*

{{ most_recent_post.content }}

---

  </div>

  <aside style="flex:1; min-width:220px; padding-left:1rem; border-left:2px solid #ff2bd3; font-size:0.9em;">

    <h2 style="margin-top:0; color:#ff2bd3;">All Posts</h2>

    <ul style="list-style:none; padding-left:0;">

    {% for post in site.posts %}
      <li style="margin-bottom:0.75em;">
        <a href="{{ post.url | relative_url }}"
           style="color:#ff2bd3; text-decoration:none; font-weight:600;">
          {{ post.title }}
        </a><br>
        <span style="font-size:0.8em; color:#d87abf;">
          {{ post.date | date: "%b %d, %Y" }}
        </span>
      </li>
    {% endfor %}

    </ul>

  </aside>

</div>
