---
layout: base
title: Background with Object
description: Use JavaScript to have an in motion background.
sprite: images/platformer/sprites/chiikawa.png
background: images/platformer/backgrounds/chiikawa-bg.jpeg
permalink: /background2
---

<canvas id="world"></canvas>

<script>
  // gets and draws canvas
  const canvas = document.getElementById("world");
  const ctx = canvas.getContext('2d');

  // loads bg and sprite images
  const backgroundImg = new Image();
  const spriteImg = new Image();
  backgroundImg.src = '{{page.background}}'; // background path from front matter
  spriteImg.src = '{{page.sprite}}';         // sprite path from front matter

  // tracks when images are ready
  let imagesLoaded = 0;

  // when background loads, it starts the game
  backgroundImg.onload = function() {
    imagesLoaded++;
    startGameWorld();
  };

  // when sprite loads, it starts the game
  spriteImg.onload = function() {
    imagesLoaded++;
    startGameWorld();
  };

  // called when both images are loaded
  function startGameWorld() {
    if (imagesLoaded < 2) return; 

    // draws object onto the canvas
    class GameObject {
      constructor(image, width, height, x = 0, y = 0, speedRatio = 0) {
        this.image = image;
        this.width = width;
        this.height = height;
        this.x = x;
        this.y = y;
        this.speedRatio = speedRatio;              // proportion of global speed to use
        this.speed = GameWorld.gameSpeed * this.speedRatio;
      }
      update() {} // placeholder for child classes
      draw(ctx) {
        ctx.drawImage(this.image, this.x, this.y, this.width, this.height);
      }
    }

    // background class that scrolls horizontally
    class Background extends GameObject {
      constructor(image, gameWorld) {
        // scrolls the entire canvas
        super(image, gameWorld.width, gameWorld.height, 0, 0, 0.1);
      }
      update() {
        // moves background to the left, makes a background loop
        this.x = (this.x - this.speed) % this.width;
      }
      draw(ctx) {
        // makes the background twice so that it's seemless in transitioning.
        ctx.drawImage(this.image, this.x, this.y, this.width, this.height);
        ctx.drawImage(this.image, this.x + this.width, this.y, this.width, this.height);
      }
    }

    // creates the player
    class Player extends GameObject {
      constructor(image, gameWorld) {
        // scales the sprite times three
        const width = image.naturalWidth * 3;
        const height = image.naturalHeight * 3;
        // centers player
        const x = (gameWorld.width - width) / 2;
        const y = (gameWorld.height - height) / 2;
        super(image, width, height, x, y);
        this.baseY = y;     // saves original Y position
        this.frame = 0;     // frames counter for sine wave animation
      }
      update() {
        // Float up and down smoothly using sine wave
        this.y = this.baseY + Math.sin(this.frame * 0.05) * 20;
        this.frame++;
      }
    }

    // creates the game itself.
    class GameWorld {
      static gameSpeed = 5; // modifies the speed of the game

      constructor(backgroundImg, spriteImg) {
        // canvas dimensons fit the window
        this.canvas = document.getElementById("world");
        this.ctx = this.canvas.getContext('2d');
        this.width = window.innerWidth;
        this.height = window.innerHeight;
        this.canvas.width = this.width;
        this.canvas.height = this.height;

        // makes the canvas fit full screen
        this.canvas.style.width = `${this.width}px`;
        this.canvas.style.height = `${this.height}px`;
        this.canvas.style.position = 'absolute';
        this.canvas.style.left = `0px`;
        this.canvas.style.top = `${(window.innerHeight - this.height) / 2}px`;

        // add game objects: background + player
        this.objects = [
         new Background(backgroundImg, this),
         new Player(spriteImg, this)
        ];
      }

      // loops game by drawing objects
      gameLoop() {
        this.ctx.clearRect(0, 0, this.width, this.height); // clears canvas
        for (const obj of this.objects) {
          obj.update();   // updates position and animation
          obj.draw(this.ctx); // draws the sprite in game
        }
        requestAnimationFrame(this.gameLoop.bind(this)); // loops sprite
      }

      // starts the loop
      start() {
        this.gameLoop();
      }
    }

    // creates and starts the game using the background and sprite images
    const world = new GameWorld(backgroundImg, spriteImg);
    world.start();
  }
</script>