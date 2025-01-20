# Most Recent Post

{% assign most_recent_post = site.posts.first %}

## [{{ most_recent_post.title }}]({{ most_recent_post.url | relative_url }})
*{{ most_recent_post.date | date: "%B %d, %Y"}}*

{{ most_recent_post.excerpt }}

[Read more]({{ most_recent_post.url | relative_url }})

---

# All Posts

{% for post in site.posts offset:1 %}
## [{{ post.title }}]({{ post.url | relative_url }})
*{{ post.date | date: "%B %d, %Y"}}*

{{ post.excerpt }}

[Read more]({{ post.url | relative_url }})

---
{% endfor %}

# Contact Me

- [LinkedIn](https://www.linkedin.com/in/your-profile)
- [Email](mailto:your-email@example.com)
