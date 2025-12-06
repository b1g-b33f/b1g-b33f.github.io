<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">

  <!-- MOBILE FIX (required) -->
  <meta name="viewport" content="width=device-width, initial-scale=1">

  <title>{{ page.title | default: site.title }}</title>

  {{ site.head }}

  <style>
    * {
      box-sizing: border-box;
    }

    html, body {
      margin: 0;
      padding: 0;
      max-width: 100%;
      overflow-x: hidden;
    }

    img {
      max-width: 100%;
      height: auto;
      display: block;
    }

    pre {
      max-width: 100%;
      overflow-x: auto;
      padding: 0.75rem;
      background: #1a1a1a;
      border-radius: 6px;
    }

    .layout {
      display: flex;
      gap: 2rem;
      align-items: flex-start;
      flex-wrap: wrap;
      max-width: 1100px;
      margin: 0 auto;
      padding: 0 1rem;
      width: 100%;
    }

    .layout-main {
      flex: 3 1 0;
      min-width: 0;
    }

    .sidebar {
      flex: 1 1 0;
      max-width: 100%;
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
    }

    .sidebar-date {
      font-size: 0.8em;
      color: #d87abf;
    }

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
</head>

<body>

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
   style="padding:6px 12px; border:1px solid #ff2bd3; border-radius:4px; text-decoration:none; color:#ff2bd3;">
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
          <span class="sidebar-date">{{ post.date | date: "%b %d, %Y" }}</span>
        </li>
      {% endfor %}
      </ul>
    </aside>

  </div>

</body>
</html>

