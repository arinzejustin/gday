<script lang="ts">
  import { onMount } from "svelte";
  import gsap from "gsap";

  const musicUrl = "/Smile-for-me.mp3"; // Place in public/music/
  let audio: HTMLAudioElement | null = null;

  let showEnvelope = true;
  let showMessage = false;
  let showButtons = false;
  let showModal = false;
  let modalMessage = "";
  let selectedChoice: "blue" | "red" | null = null;

  // Check localStorage for previous choice
  onMount(() => {
    if (!audio) {
      audio = new Audio(musicUrl);
      audio.loop = true;
      audio.volume = 0.4;
    }
    
    const savedChoice = localStorage.getItem("pillChoice");
    if (savedChoice) {
      selectedChoice = savedChoice as "blue" | "red";
      showEnvelope = false;
      showMessage = true;
      showButtons = false;
      modalMessage = `You previously chose the ${savedChoice} pill.`;
      showModal = true;
    }
    cycleTitle();
    createBubbles();
    window.addEventListener("click", handleClick);
    document.body.addEventListener("touchstart", handleTouch);

    return () => {
      window.removeEventListener("click", handleClick);
      document.body.removeEventListener("touchstart", handleTouch);
    };
  });

  // Alternating title words
  const titleWords = [
    "Please Read Me Momma",
    "Please Open Me My Queen",
    "Open Me My Love",
    "Open Me My Darling",
  ];
  let displayTitle = "";
  let titleIndex = 0;
  let charIndex = 0;
  let deleting = false;

  // Love letter
  const letter = `Happy Girlfriend's Day, my love ❤️

From the moment you walked into my life, everything changed for the better.
You’ve filled my days with laughter, my heart with warmth, and my soul with peace.

Forever yours,
[Your Name]`;

  let displayedText = "";
  let cursor = "❤️";
  let letterIndex = 0;

  async function handlePillChoice(choice: "blue" | "red") {
    if (!selectedChoice) {
      try {
        const response = await fetch("/api/record-choice", {
          method: "POST",
          headers: {
            "Content-Type": "application/json",
          },
          body: JSON.stringify({ choice }),
        });

        if (!response.ok) {
          throw new Error(`Request failed with status ${response.status}`);
        }

        selectedChoice = choice;
        localStorage.setItem("pillChoice", choice);
        showButtons = false;
        modalMessage = `Your ${choice} pill choice has been saved! ❤️`;
        showModal = true;
      } catch (error) {
        modalMessage = `Oops, something went wrong. Please try again.`;
        showModal = true;
      }
    }
  }

  function closeModal() {
    showModal = false;
  }

  function cycleTitle() {
    const word = titleWords[titleIndex];
    if (!deleting && charIndex < word.length) {
      displayTitle += word[charIndex++];
      setTimeout(cycleTitle, 300);
    } else if (deleting && charIndex > 0) {
      displayTitle = word.substring(0, --charIndex);
      setTimeout(cycleTitle, 100);
    } else {
      if (!deleting) {
        deleting = true;
        setTimeout(cycleTitle, 1000);
      } else {
        deleting = false;
        titleIndex = (titleIndex + 1) % titleWords.length;
        setTimeout(cycleTitle, 500);
      }
    }
  }

  function openEnvelope() {
    if (!audio) {
      audio = new Audio(musicUrl);
      audio.loop = true;
      audio.volume = 0.4;
      audio
        .play()
        .catch(() => console.log("Autoplay blocked until interaction"));
    } else {
      audio.volume = 0.4;
      audio
        .play()
        .catch(() => console.log("Autoplay blocked until interaction"));
    }
    showEnvelope = false;
    createHeartAnimation();
  }

  function createHeartAnimation() {
    let heart = document.createElement("div");
    heart.innerHTML = "❤️";
    heart.className = "center-heart";
    document.body.appendChild(heart);

    const maxDimension = Math.max(window.innerWidth, window.innerHeight);
    const targetScale = maxDimension / 30;

    gsap.fromTo(
      heart,
      {
        scale: 0,
        opacity: 1,
        x: window.innerWidth / 2,
        y: window.innerHeight / 2,
      },
      {
        scale: targetScale,
        opacity: 0.8,
        duration: 1.5,
        ease: "power2.out",
        onComplete: () => {
          for (let i = 0; i < 20; i++) {
            let burstHeart = document.createElement("div");
            burstHeart.innerHTML = "❤️";
            burstHeart.className = "burst-heart";
            document.body.appendChild(burstHeart);

            const angle = (i / 20) * 4 * Math.PI;
            const distance = Math.random() * 100 + 50;

            gsap.set(burstHeart, {
              x: window.innerWidth / 2,
              y: window.innerHeight / 2,
              scale: 0.5,
              opacity: 1,
            });

            gsap.to(burstHeart, {
              x: window.innerWidth / 2 + Math.cos(angle) * distance,
              y: window.innerHeight / 2 + Math.sin(angle) * distance,
              scale: 1.5,
              opacity: 0,
              duration: 0.8,
              ease: "power2.out",
              delay: Math.random() * 0.3,
              onComplete: () => burstHeart.remove(),
            });
          }

          gsap.to(heart, {
            opacity: 0,
            duration: 0.5,
            onComplete: () => {
              heart.remove();
              showMessage = true;
              typeLetter();
            },
          });
        },
      },
    );
  }

  function typeLetter() {
    if (letterIndex < letter.length) {
      displayedText += letter[letterIndex++];
      setTimeout(typeLetter, 40);
    } else {
      showButtons = true;
      finalRomanticEnding();
    }
  }

  function finalRomanticEnding() {
    if (audio) {
      gsap.to(audio, {
        volume: 0,
        duration: 4,
        onComplete: () => audio?.pause(),
      });
    }

    setTimeout(() => {
      const lastLine = document.querySelector(".final-line") as HTMLElement;
      if (lastLine) {
        gsap.to(lastLine, { scale: 1.1, repeat: 5, yoyo: true, duration: 0.5 });
      }
    }, 500);

    for (let i = 0; i < 40; i++) {
      let particle = document.createElement("div");
      particle.innerHTML = Math.random() > 0.5 ? "❤️" : "✨";
      particle.className = "falling-particle";
      document.body.appendChild(particle);

      gsap.set(particle, {
        x: Math.random() * window.innerWidth,
        y: -1000,
        fontSize: `${Math.random() * 20 + 15}px`,
        opacity: Math.random() * 0.5 + 0.5,
      });

      gsap.to(particle, {
        y: window.innerHeight + 50,
        x: `+=${(Math.random() - 0.5) * 100}`,
        duration: Math.random() * 3 + 3,
        ease: "linear",
        onComplete: () => {
          particle.remove();
        },
      });
    }
  }

  function rippleHeartsAt(x: number, y: number) {
    for (let i = 0; i < 8; i++) {
      let heart = document.createElement("div");
      heart.innerHTML = "❤️";
      heart.className = "ripple-heart";
      document.body.appendChild(heart);

      gsap.set(heart, { x, y, scale: 0 });
      gsap.to(heart, {
        scale: Math.random() * 1 + 0.3,
        x: x + (Math.random() - 0.5) * 150,
        y: y + (Math.random() - 0.5) * 150,
        opacity: 0,
        duration: 1.2,
        ease: "power2.out",
        onComplete: () => heart.remove(),
      });
    }
  }

  function handleClick(e: MouseEvent) {
    rippleHeartsAt(e.clientX, e.clientY);
  }

  function handleTouch(e: TouchEvent) {
    const touch = e.touches[0];
    rippleHeartsAt(touch.clientX, touch.clientY);
  }

  function createBubbles() {
    const bubbleContainer = document.createElement("div");
    bubbleContainer.className = "bubble-container";
    document.body.appendChild(bubbleContainer);

    const numBubbles = 40;
    for (let i = 0; i < numBubbles; i++) {
      let bubble = document.createElement("div");
      bubble.className = "bubble";
      bubbleContainer.appendChild(bubble);

      const size = Math.random() * 25 + 10;
      const startX = Math.random() * window.innerWidth;
      const startY = Math.random() * window.innerHeight;
      const zIndex = Math.floor(Math.random() * 5);

      gsap.set(bubble, {
        width: size,
        height: size,
        x: startX,
        y: startY,
        opacity: 0,
        zIndex: zIndex,
      });

      const tl = gsap.timeline({ repeat: -1, yoyo: true });
      tl.to(bubble, {
        opacity: Math.random() * 0.3 + 0.2,
        duration: 1,
        ease: "power1.in",
      })
        .to(
          bubble,
          {
            x: `+=${(Math.random() - 0.5) * 300}`,
            y: `+=${(Math.random() - 0.5) * 300}`,
            scale: Math.random() * 0.7 + 0.8,
            rotation: Math.random() * 30 - 15,
            duration: Math.random() * 6 + 6,
            ease: "sine.inOut",
          },
          0,
        )
        .to(
          bubble,
          {
            opacity: Math.random() * 0.2 + 0.1,
            duration: 1,
            ease: "power1.out",
          },
          "-=1",
        );

      gsap.to(bubble, {
        x: `+=${Math.sin(i) * 20}`,
        y: `+=${Math.cos(i) * 20}`,
        duration: Math.random() * 2 + 1,
        ease: "sine.inOut",
        repeat: -1,
        yoyo: true,
        delay: Math.random() * 2,
      });
    }

    const handleResize = () => {
      const bubbles = bubbleContainer.querySelectorAll(".bubble");
      bubbles.forEach((bubble) => {
        gsap.set(bubble, {
          x: Math.random() * window.innerWidth,
          y: Math.random() * window.innerHeight,
        });
      });
    };

    window.addEventListener("resize", handleResize);
    return () => {
      window.removeEventListener("resize", handleResize);
      bubbleContainer.remove();
    };
  }
</script>

<link rel="stylesheet" href="/styles.css" />

{#if showEnvelope}
  <div class="center">
    <div class="title-text">{displayTitle}</div>
    <!-- svelte-ignore a11y_click_events_have_key_events -->
    <!-- svelte-ignore a11y_no_static_element_interactions -->
    <svg
      class="envelope-svg"
      viewBox="0 0 512 512"
      fill="#fff"
      on:click={openEnvelope}
    >
      <path
        d="M502.3 190.8L327.4 338.1c-28.5 23.6-70.3 23.6-98.8 0L9.7 190.8C3.9 186.2 0 179.1 0 171.5V120c0-13.3 10.7-24 24-24h464c13.3 0 24 10.7 24 24v51.5c0 7.6-3.9 14.7-9.7 19.3z"
      />
      <path
        d="M0 216v184c0 13.3 10.7 24 24 24h464c13.3 0 24-10.7 24-24V216L327.4 363.3c-28.5 23.6-70.3 23.6-98.8 0L0 216z"
      />
    </svg>
  </div>
{/if}

{#if showMessage}
  <div class="message">
    {@html displayedText.replace(
      /(Forever yours,.*$)/,
      '<span class="final-line">$1</span>',
    )}
    <span class="cursor">{cursor}</span>
  </div>
{/if}

{#if showButtons}
  <div class="button-container">
    <button
      class="pill-button blue-pill"
      on:click={() => handlePillChoice("blue")}>Blue Pill</button
    >
    <button
      class="pill-button red-pill"
      on:click={() => handlePillChoice("red")}>Red Pill</button
    >
  </div>
{/if}

{#if showModal}
  <div class="modal-overlay">
    <div class="modal">
      <p>{modalMessage}</p>
      <button class="modal-close" on:click={closeModal}>Close</button>
    </div>
  </div>
{/if}
