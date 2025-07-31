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
  const letter = `Happy Girlfriend's Day, my love ❤️🌹

From the very moment you walked into my life, everything changed for the better 💖.  
Before you, love was just a word I thought I understood… but you showed me its meaning in the most beautiful way ✨.  
You are not just my girlfriend — you are my heart, my safe place, my answered prayer 🙏💞.  
Every smile you give me is like a sunrise 🌅 that lights up my entire world, and every touch of your hand feels like home 🏡❤️.

Deep in my soul, there is a dream I carry every single day… the dream of you becoming my wife 💍❤️.  
I imagine it so clearly 😍 — the day we stand together at the altar, you in the most beautiful dress 👰, me barely able to hold back my tears 🥹, my heart pounding with joy 💓.  
The moment you say “I do” 💌, I will know that every prayer, every hope, every wait was worth it 🌈.  
That moment will be the start of our forever — a forever I have longed for since the very first time I saw you 🌟.

I dream of our home together 🏡, filled with laughter 😂, warm hugs 🤗, and the sound of little feet running through the halls 👣.  
I dream of us raising our children together 🍼❤️ — you holding our first baby while I look at you and think, *"Wow… this is the woman I get to spend my life with."* 🥰  
I imagine family dinners 🍽️, bedtime stories 📖, playful arguments over what movie to watch 🎬, and us smiling at each other from across the room, knowing in our hearts… *we made this life together* 💞.

But my dreams go even further than that 🌏💫… 
I dream of traveling the world with you ✈️🌍, seeing new places and making memories that will last forever 🗺️❤️.  
I dream of us walking on beaches hand in hand 🏖️, watching sunsets 🌅, laughing under the stars ✨, and taking silly pictures just to remember how happy we felt.  
I dream of us growing old together 👵👴, sitting on the porch one day, holding hands 🤝, and smiling as we watch our grandchildren play 🍼💞.

Through every challenge we’ve faced 💔, you’ve shown me what real love is 💗.  
You’ve forgiven me even when I didn’t deserve it 😔, you’ve chosen us even when things were hard 💪💞.  
Your heart is the most beautiful thing about you ❤️ — and I promise I will never take it for granted.  
I will never forsake you 🚫💔, I will never leave you 🚶‍♂️❌, and I will always love you more each day 🌹💘.  
You are my all in all, my everything, my reason to keep going 🌟.

I have never told you this before 😌… but I will never forget the very first day we met 🌙✨.  
It was at that junction at night 🌃.  
I remember my look towards you that evening 😍 — it wasn’t just attraction, it was something deeper 💫.  
It was as if my soul recognized you before my mind even understood 🌌💞.  
And then… your eyes 👀💖.  
Those charming, captivating eyes 😍💎 — I’ve never told you how much they’ve stayed with me.  
They have a light, a gentleness, and a magic that makes me feel seen, understood, and loved 💫❤️.

I love the way you laugh 😂❤️, the way you care so deeply 💞, the way you make me feel like I matter 🌹.  
I love how you encourage me when I’m down 🕊️, how you believe in me when I start to doubt myself 🙌, and how you hold my hand through every storm 🌧️☀️.  
I love your stubbornness 😄, your kindness 🪷, your strength 💪, and the way your heart always finds a way to love again 💖.

My love, I am yours completely 💍.  
My heart belongs to you 💓, my soul belongs to you 💞, and my future belongs to you 🌹.  
No matter what life throws at us 🌪️, I will be right here — holding your hand 🤝, standing by your side 💕, and loving you with everything in me ❤️🔥.

When I think of the future, I don’t see a timeline or a plan — I see *you* 😍.  
I see your smile lighting up every day of my life 🌅💞.  
I see your voice being the sweetest sound I’ll ever hear 🎶❤️.  
I see your love as the anchor that will keep me steady no matter what comes 🌊⚓.

You are the best part of me 🥰.  
You inspire me to be better 🌟, to dream bigger 🌈, and to love deeper 💖.  
Every single day, I thank God for you 🙏, for your love 💞, for your laughter 😂, for your patience 😌, and for your beautiful soul 💐.

Happy Girlfriend’s Day, my queen 👑❤️.  
You are my forever, my dream come true 💖, my one and only 💍💞.  
And one day soon… I will see you walking towards me at the altar 💒, and I will whisper in my heart… *I finally get to call her my wife* 💘💍.

Forever yours,  
Okechukwu Justin Arinze ❤️🌹💌`;


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
    const targetScale = maxDimension / 18;

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
      const currentChar = letter[letterIndex++];
      displayedText += currentChar;

      // Default typing speed
      let delay = 50;

      // Add short pause after sentence endings
      if (currentChar === "." || currentChar === "!" || currentChar === "?") {
        delay = 400;
      }

      // Add longer pause after emotional beats
      if (
        currentChar === "❤️" ||
        currentChar === "💍" ||
        (currentChar === "\n" && letter[letterIndex] === "\n")
      ) {
        delay = 1200;
      }

      setTimeout(typeLetter, delay);
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
