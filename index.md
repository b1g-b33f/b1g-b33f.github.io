![Banner](https://github.com/user-attachments/assets/bbe184bd-d1b3-46e1-aaa6-9a7c59edfd35)






Welcome to the blog! I'm Shawn, a former car wash manager now working in the world of offensive security. I'm hoping to share some of my journey and what I've learned along the way. I'm currently focused on web app pentesting and application security but constantly learning other areas of pentesting.

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

- [LinkedIn](https://www.linkedin.com/in/shawn-szczepkowski/)
- [Email](mailto:shawnszczepkowski@gmail.com)
