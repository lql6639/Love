<script setup>
import { onBeforeUnmount, onMounted } from 'vue'
import musicnotation from '../public/rotate.png'

import config from '@/../package.json'

let onResizeHandler = null

onMounted(() => {
  // 点击图片实现功能
  function runing() {
    // 设置audio初始音量
    const music = document.getElementById('music')
    const rotate = document.getElementById('rotate')
    music.volume = 0.8

    // 点击图片实现播放音乐或不播放音乐和实现图片一直旋转或停止旋转
    let playing = false

    rotate.addEventListener('click', () => {
      if (!playing) {
        // 播放
        music.play()
        // 旋转
        playing = true
        rotate.style.animation = 'rotation 4s linear infinite'
      } else {
        // 不播放
        music.pause()
        // 不旋转
        playing = false
        rotate.style.webkitAnimationPlayState = 'paused'
      }
    })

    // 添加audio对play事件的监听器
    music.addEventListener('play', () => {
      playing = true
      rotate.style.animation = 'rotation 4s linear infinite'
    })

    // 添加audio对pause事件的监听器
    music.addEventListener('pause', () => {
      playing = false
      rotate.style.webkitAnimationPlayState = 'paused'
    })
  }
  runing()

    // 控制台输出
  const styleTitle1 = 'font-size: 20px;font-weight: 600;color: rgb(244,167,89);'
  const styleTitle2 = 'font-size:12px;color: rgb(244,167,89);'
  const styleContent = 'color: rgb(30,152,255);'
  const title1 = 'Lovlin_0206 の Love'
  const title2 = `
    __  __  _  _____  _____  _   _   ____ 
  |  \\/  |(_)|  ___|| ____|| \\ | | / ___|
  | |\\/| || || |_   |  _|  |  \\| || |  _ 
  | |  | || ||  _|  | |___ | |\\  || |_| |
  |_|  |_||_||_|    |_____||_| \\_| \\____|`
  const content = `\n\n版本: ${config.version}\nAuthor: ${config.author} & Github: ${config.AuthorGithub}`
  console.info(
    `%c${title1} %c${title2} %c${content}`,
    styleTitle1,
    styleTitle2,
    styleContent
  )

  // Settings
  const settings = {
    particles: {
      // maximum amount of particles
      length: 500,
      // particle duration in sec
      duration: 2,
      // particle velocity in pixels/sec
      velocity: 100,
      // play with this for a nice effect
      effect: -0.75,
      // particle size in pixels
      size: 30,
    },
  }

  // RequestAnimationFrame polyfill by Erik Möller
  ;(function () {
    let b = 0
    const c = ['ms', 'moz', 'webkit', 'o']
    for (let a = 0; a < c.length && !window.requestAnimationFrame; ++a) {
      window.requestAnimationFrame = window[c[a] + 'RequestAnimationFrame']
      window.cancelAnimationFrame =
        window[c[a] + 'CancelAnimationFrame'] ||
        window[c[a] + 'CancelRequestAnimationFrame']
    }
    if (!window.requestAnimationFrame) {
      window.requestAnimationFrame = function (h, e) {
        const d = new Date().getTime()
        const f = Math.max(0, 16 - (d - b))
        const g = window.setTimeout(() => h(d + f), f)
        b = d + f
        return g
      }
    }
    if (!window.cancelAnimationFrame) {
      window.cancelAnimationFrame = function (d) {
        clearTimeout(d)
      }
    }
  })()

  // Point class
  const Point = (function () {
    function Point(x, y) {
      this.x = typeof x !== 'undefined' ? x : 0
      this.y = typeof y !== 'undefined' ? y : 0
    }
    Point.prototype.clone = function () {
      return new Point(this.x, this.y)
    }
    Point.prototype.length = function (length) {
      if (typeof length === 'undefined') return Math.sqrt(this.x * this.x + this.y * this.y)
      this.normalize()
      this.x *= length
      this.y *= length
      return this
    }
    Point.prototype.normalize = function () {
      const len = this.length()
      this.x /= len
      this.y /= len
      return this
    }
    return Point
  })()

  // Particle class
  const Particle = (function () {
    function Particle() {
      this.position = new Point()
      this.velocity = new Point()
      this.acceleration = new Point()
      this.age = 0
    }
    Particle.prototype.initialize = function (x, y, dx, dy) {
      this.position.x = x
      this.position.y = y
      this.velocity.x = dx
      this.velocity.y = dy
      this.acceleration.x = dx * settings.particles.effect
      this.acceleration.y = dy * settings.particles.effect
      this.age = 0
    }
    Particle.prototype.update = function (deltaTime) {
      this.position.x += this.velocity.x * deltaTime
      this.position.y += this.velocity.y * deltaTime
      this.velocity.x += this.acceleration.x * deltaTime
      this.velocity.y += this.acceleration.y * deltaTime
      this.age += deltaTime
    }
    Particle.prototype.draw = function (context, image) {
      function ease(t) {
        return --t * t * t + 1
      }
      const size = image.width * ease(this.age / settings.particles.duration)
      context.globalAlpha = 1 - this.age / settings.particles.duration
      context.drawImage(
        image,
        this.position.x - size / 2,
        this.position.y - size / 2,
        size,
        size,
      )
    }
    return Particle
  })()

  // ParticlePool class
  const ParticlePool = (function () {
    const particles = []
    let firstActive = 0
    let firstFree = 0
    const duration = settings.particles.duration

    function ParticlePool(length) {
      // create and populate particle pool
      for (let i = 0; i < length; i++) particles.push(new Particle())
    }
    ParticlePool.prototype.add = function (x, y, dx, dy) {
      particles[firstFree].initialize(x, y, dx, dy)
      // handle circular queue
      firstFree++
      if (firstFree === particles.length) firstFree = 0
      if (firstActive === firstFree) firstActive++
      if (firstActive === particles.length) firstActive = 0
    }
    ParticlePool.prototype.update = function (deltaTime) {
      // update active particles
      let i
      if (firstActive < firstFree) {
        for (i = firstActive; i < firstFree; i++) particles[i].update(deltaTime)
      }
      if (firstFree < firstActive) {
        for (i = firstActive; i < particles.length; i++) particles[i].update(deltaTime)
        for (i = 0; i < firstFree; i++) particles[i].update(deltaTime)
      }
      // remove inactive particles
      while (particles[firstActive].age >= duration && firstActive !== firstFree) {
        firstActive++
        if (firstActive === particles.length) firstActive = 0
      }
    }
    ParticlePool.prototype.draw = function (context, image) {
      // draw active particles
      let i
      if (firstActive < firstFree) {
        for (i = firstActive; i < firstFree; i++) particles[i].draw(context, image)
      }
      if (firstFree < firstActive) {
        for (i = firstActive; i < particles.length; i++) particles[i].draw(context, image)
        for (i = 0; i < firstFree; i++) particles[i].draw(context, image)
      }
    }
    return ParticlePool
  })()

  // Putting it all together
  ;(function (canvas) {
    const context = canvas.getContext('2d')
    const particles = new ParticlePool(settings.particles.length)
    // particles/sec
    const particleRate = settings.particles.length / settings.particles.duration
    let time
    // get point on heart with -PI <= t <= PI
    function pointOnHeart(t) {
      return new Point(
        160 * Math.pow(Math.sin(t), 3),
        130 * Math.cos(t) -
          50 * Math.cos(2 * t) -
          20 * Math.cos(3 * t) -
          10 * Math.cos(4 * t) +
          25,
      )
    }
    // creating the particle image using a dummy canvas
    const image = (function () {
      const canvas = document.createElement('canvas')
      const context = canvas.getContext('2d')
      canvas.width = settings.particles.size
      canvas.height = settings.particles.size
      // helper function to create the path
      function to(t) {
        const point = pointOnHeart(t)
        point.x =
          settings.particles.size / 2 +
          (point.x * settings.particles.size) / 350
        point.y =
          settings.particles.size / 2 -
          (point.y * settings.particles.size) / 350
        return point
      }
      // create the path
      context.beginPath()
      let t = -Math.PI
      let point = to(t)
      context.moveTo(point.x, point.y)
      while (t < Math.PI) {
        t += 0.01 // baby steps!
        point = to(t)
        context.lineTo(point.x, point.y)
      }
      context.closePath()
      // create the fill
      context.fillStyle = 'red'
      context.fill()
      // create the image
      const image = new Image()
      image.src = canvas.toDataURL()
      return image
    })()
    // render that thing!
    function render() {
      // next animation frame
      requestAnimationFrame(render)
      // update time
      const newTime = new Date().getTime() / 1000
      const deltaTime = newTime - (time || newTime)
      time = newTime
      // clear canvas
      context.clearRect(0, 0, canvas.width, canvas.height)
      // create new particles
      const amount = particleRate * deltaTime
      for (let i = 0; i < amount; i++) {
        const pos = pointOnHeart(Math.PI - 2 * Math.PI * Math.random())
        const dir = pos.clone().length(settings.particles.velocity)
        particles.add(
          canvas.width / 2 + pos.x,
          canvas.height / 2 - pos.y,
          dir.x,
          -dir.y,
        )
      }
      // update and draw particles
      particles.update(deltaTime)
      particles.draw(context, image)
    }
    // handle (re-)sizing of the canvas
    function onResize() {
      canvas.width = canvas.clientWidth
      canvas.height = canvas.clientHeight
    }
    onResizeHandler = onResize
    window.addEventListener('resize', onResize)
    // delay rendering bootstrap
    setTimeout(() => {
      onResize()
      render()
    }, 10)
  })(document.getElementById('heart'))
})

onBeforeUnmount(() => {
  if (onResizeHandler) window.removeEventListener('resize', onResizeHandler)
})
</script>

<template>
  <Header>
    <!-- 旋转播放 -->
    <div class="graph" id="graph">
      <img class="rotate" id="rotate" :src="musicnotation" />
    </div>
    <audio src="https://api.qijieya.cn/meting/?type=url&id=1359356908" id="music" loop></audio>
  </Header>
  
  <main>
    <!-- 爱心 -->
    <canvas id="heart"></canvas>
  </main>
</template>
