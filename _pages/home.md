---
layout: splash
permalink: /
header:
  image: cover.jpg
feature_row:
  - image_path: spectro_output.png
    alt: "Sample Spectra"
    title: "Raman Spectrometer"
    excerpt: ""
    btn_label: "Learn More"
  - image_path: mm-responsive-feature.png
    alt: "fully responsive"
    title: "2 Meter Repeater Rebuild"
    excerpt: "Built on HTML5 + CSS3. All layouts are fully responsive with helpers to augment your content."
    url: "/docs/layouts/"
    btn_label: "Learn More"
  - image_path: mm-free-feature.png
    alt: "100% free"
    title: "100% Free"
    excerpt: "Free to use however you want under the MIT License. Clone it, fork it, customize it, whatever!"
    url: "/docs/license/"
    btn_label: "Learn More"
intro:
  - excerpt: 'Missouri S&T Mars Rover Design Team
  2017 Autonomous Systems Lead & Professional FPGA Engineer'
---

{% include feature_row id="intro" type="center" %}

<div class="grid__wrapper">
  {% for post in site.projects  %}
    {% include archive-single.html type="grid" %}
  {% endfor %}
</div>