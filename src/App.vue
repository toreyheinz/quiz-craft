<script setup>
</script>

<template>
  <div id="app">
    <header class="app-header">
      <router-link to="/" class="header-content">
        <img 
          src="/quiz-craft-logo.jpg" 
          alt="Quiz Craft Logo" 
          class="header-logo"
        />
        <h1 class="header-title">Quiz Craft</h1>
      </router-link>
    </header>
    <main class="app-main">
      <router-view />
    </main>
  </div>
</template>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, 'Open Sans', 'Helvetica Neue', sans-serif;
  background-color: #f5f5f5;
  color: #333;
}

#app {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
}

.app-header {
  background: white;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
  position: sticky;
  top: 0;
  z-index: 100;
}

.header-content {
  display: flex;
  align-items: center;
  gap: 1rem;
  text-decoration: none;
  color: inherit;
  max-width: 800px;
  margin: 0 auto;
  padding: 1rem 2rem;
}

.header-logo {
  height: 48px;
  border-radius: 5px;
  box-shadow: 0 2px 8px rgba(0,0,0,0.15);
  transition: transform 0.2s, box-shadow 0.2s;
}

.header-content:hover .header-logo {
  transform: translateY(-1px);
  box-shadow: 0 4px 12px rgba(0,0,0,0.2);
}

.header-title {
  font-size: 1.5rem;
  color: #2c3e50;
  font-weight: 600;
  margin: 0;
}

.app-main {
  flex: 1;
}

@media (max-width: 768px) {
  .header-content {
    padding: 0.75rem 1rem;
  }
  
  .header-logo {
    height: 40px;
  }
  
  .header-title {
    font-size: 1.25rem;
  }
}
</style>