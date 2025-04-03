---
# https://vitepress.dev/reference/default-theme-home-page
layout: home

titleTemplate: "Startseite"

hero:
  name: "NixOS & Flakes"
  text: "Eine unoffizielle Zusammenstellung für Anfänger"
  tagline:
    Möchtest du NixOS & Flakes im Detail kennen lernen? Suchst du nach einem anfängerfreundlichen
    Tutorial? Dann bist du hier genau richtig!
  image:
    src: /logo.png
    alt: NixOS Flakes
  actions:
    - theme: brand
      text: Vorwort
      link: /preface.md
    - theme: brand
      text: Anfänge
      link: /introduction/index.md
    - theme: alt
      text: Auf GitHub anzeigen
      link: https://github.com/ryan4yin/nixos-and-flakes-book
# features:
#   - title: Feature A
#     details: Lorem ipsum dolor sit amet, consectetur adipiscing elit
#   - title: Feature B
#     details: Lorem ipsum dolor sit amet, consectetur adipiscing elit
#   - title: Feature C
#     details: Lorem ipsum dolor sit amet, consectetur adipiscing elit
---

<style>
:root {
  --vp-home-hero-name-color: transparent;
  --vp-home-hero-name-background: -webkit-linear-gradient(120deg, #4565d8 40%, #41d1ff);

  --vp-home-hero-image-background-image: linear-gradient(-60deg, #41d1ff 40%, #0fffc1);
}

.image-bg {
  transform: scale(0.75);
  -webkit-filter: blur(5vw);
  -moz-filter: blur(5vw);
  -ms-filter: blur(5vw);
  filter: blur(5vw);
  background-size: 200% 200%;
  animation: animateGlow 10s ease infinite;
}

@keyframes animateGlow {
  0% {
    background-position: 0% 50%;
  }
  50% {
    background-position: 100% 50%;
  }
  100% {
    background-position: 0% 50%;
  }
}

@media (min-width: 640px) {
  :root {
    --vp-home-hero-image-filter: blur(56px);
  }
}

@media (min-width: 960px) {
  :root {
    --vp-home-hero-image-filter: blur(72px);
  }
}
</style>
