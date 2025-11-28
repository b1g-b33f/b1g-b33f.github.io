<style>
  /* Make images (banner, etc.) scale nicely on small screens */
  img {
    max-width: 100%;
    height: auto;
  }

  /* Two-column layout on desktop */
  .layout {
    display: flex;
    gap: 2rem;
    align-items: flex-start;
    flex-wrap: wrap; /* allows wrapping instead of forcing overflow */
  }

  .layout-main {
    flex: 3 1 0;
    min-width: 0;
  }

  .sidebar {
    flex: 1 1 0;
    font-size: 0.9em;
    padding-left: 1rem;
    border-left: 2px solid #ff2bd3;
  }

  .sidebar ul {
    list-style: none;
    padding-left: 0;
    margin: 0;
  }

  .sidebar li {
    margin-bottom: 0.75em;
  }

  .sidebar a {
    color: #ff2bd3;
    text-decoration: none;
    font-weight: 600;
    cursor: pointer;
  }

  .sidebar-date {
    font-size: 0.8em;
    color: #d87abf;
  }

  /* On small screens, stack main + sidebar vertically */
  @media (max-width: 800px) {
    .layout {
      flex-direction: column;
    }
    .sidebar {
      border-left: none;
      border-top: 2px solid #ff2bd3;
      padding-left: 0;
      padding-top: 1rem;
      margin-top: 1.5rem;
    }
  }
</style>

<div class="layout">

  <!-- MAIN CONTENT -->
  <div class="layout-main" markdown="1">

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

  <!-- SIDEBAR -->
  <aside class="sidebar">

    <h2 style="margin-top:0; color:#ff2bd3;">All Posts</h2>

    <ul>
    {% for post in site.posts %}
      <li>
        <a href="{{ post.url | relative_url }}">
          {{ post.title }}
        </a><br>
        <span class="sidebar-date">
          {{ post.date | date: "%b %d, %Y" }}
        </span>
      </li>
    {% endfor %}
    </ul>

  </aside>

</div>
