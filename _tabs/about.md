---
# the default layout is 'page'
icon: fas fa-info-circle
order: 6
---

Welcome to **amvatlabs** - a place where we explore, experiment, and sometimes break things… **so we don’t have to break them at work!**  

We are two recent graduates from RMIT University, passionate about cybersecurity and eager to sharpen our skills. While hunting for our first professional roles, we’ve built a home lab to experiment freely, learn hands-on, and tackle real-world cybersecurity challenges. From TryHackMe exercises to configuring networks and testing security tools, our homelab is our sandbox for learning and growth.  

Our goal is to learn by doing and to share what we learn with others who are interested in cybersecurity and hands-on lab work.

Here, you’ll find:

- Step-by-step guides to our lab setups

- Notes on tools, configurations, and troubleshooting

- Insights from our experiments and lessons learned along the way

- Occasional deep dives into cybersecurity concepts and practical applications

We believe that hands-on experience is one of the best ways to grow in cybersecurity, and we hope our journey inspires others to start their own labs, no matter how small.

---

# Meet the Team

<div class="about-authors">
{% for pair in site.data.authors %}
    {% assign author = pair[1] %}
    <div class="author-card" style="display:flex; align-items:center; margin-bottom: 1.5em; margin-top: 1.5em;">
        <img src="{{ author.avatar | relative_url }}" alt="{{ author.name }}" style="width:200px; height:200px; border-radius:50%; margin-right:40px;" loading="lazy">
        <div style="display:flex; flex-direction:column; gap:8px;">
            <h3 style="margin:0;">{{ author.name }}</h3>
            {% if author.email %}<div style="display:flex; align-items:center; gap:6px;"><i class="fas fa-envelope"></i><a href="mailto:{{ author.email }}">{{ author.email }}</a></div>{% endif %}
            <div style="display:flex; gap:16px; align-items:center; flex-wrap:wrap;">{% if author.tryhackme %}<span style="display:flex; align-items:center; gap:6px;"><span style="display:inline-block; width:20px; height:20px; background:url('https://raw.githubusercontent.com/Avinash-Mathew/assets/main/email/tryhackme.png') center/contain no-repeat;"></span><a href="{{ author.tryhackme }}" target="_blank">TryHackMe</a></span>{% endif %}{% if author.linkedin %}<span style="display:flex; align-items:center; gap:6px;"><span style="display:inline-block; width:20px; height:20px; background:url('https://raw.githubusercontent.com/Avinash-Mathew/assets/main/email/linkedin.png') center/contain no-repeat;"></span><a href="{{ author.linkedin }}" target="_blank">LinkedIn</a></span>{% endif %}{% if author.github %}<span style="display:flex; align-items:center; gap:6px;"><span style="display:inline-block; width:20px; height:20px; background:url('https://raw.githubusercontent.com/Avinash-Mathew/assets/main/email/github.png') center/contain no-repeat;"></span><a href="{{ author.github }}" target="_blank">GitHub</a></span>{% endif %}</div>
        </div>
    </div>
{% endfor %}
</div>