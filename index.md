![Banner](https://github.com/user-attachments/assets/4287d2a3-d69b-474d-9213-5c12236bd635)

Welcome to the blog! I'm Shawn, a former manager and blue collar worker now employed in the world of offensive security. I'm hoping to share some of my journey and what I've learned along the way. I'm currently focused on web app pentesting and application security but constantly learning other areas of pentesting as well.

# Most Recent Post

{% assign most_recent_post = site.posts.first %}

## [{{ most_recent_post.title }}]({{ most_recent_post.url | relative_url }})
*{{ most_recent_post.date | date: "%B %d, %Y"}}*

{{ most_recent_post.excerpt }}

[Read more]({{ most_recent_post.url | relative_url }})

---

<details>
  <summary><strong>All Posts</strong></summary>

  <ul>
  {% for post in site.posts %}
    <li>
      <a href="{{ post.url | relative_url }}">{{ post.title }}</a>
      <span> – {{ post.date | date: "%B %d, %Y" }}</span>
    </li>
  {% endfor %}
  </ul>
</details>

---

### Contact Me
- [Email](mailto:shawnszczepkowski@gmail.com)


