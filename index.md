<details style="float:left; width:250px; margin-right:20px;">
<summary><strong>All Posts</strong></summary>

{% for post in site.posts %}
- [{{ post.title }}]({{ post.url | relative_url }})
{% endfor %}

</details>

![Banner](https://github.com/user-attachments/assets/4287d2a3-d69b-474d-9213-5c12236bd635)

Welcome to the blog! I'm Shawn, a former manager and blue collar now employed in the world of offensive security. I'm hoping to share some of my journey and what I've learned along the way. I'm currently focused on web app pentesting and application security but constantly learning other areas of pentesting as well.

# Most Recent Post

{% assign most_recent_post = site.posts.first %}

## [{{ most_recent_post.title }}]({{ most_recent_post.url | relative_url }})
*{{ most_recent_post.date | date: "%B %d, %Y"}}*

{{ most_recent_post.excerpt }}

[Read more]({{ most_recent_post.url | relative_url }})

---

## Previous Posts

{% for post in site.posts offset:1 %}
## [{{ post.title }}]({{ post.url | relative_url }})
*{{ post.date | date: "%B %d, %Y"}}*

{{ post.excerpt }}

[Read more]({{ post.url | relative_url }})

---
{% endfor %}

### Contact Me
- [Email](mailto:shawnszczepkowski@gmail.com)
