<template>
  <div class="d-flex auth-container">
    <div class="image-side">
      <img src="https://images.unsplash.com/photo-1504674900247-0877df9cc836?auto=format&fit=crop&w=600&q=80"
        alt="auth form" class="auth-image" />
    </div>
    <div class="form-side">
      <div class="card border-0 h-100">
        <div class="card-body p-4 p-lg-5 text-black">
          <slot></slot>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.auth-container {
  height: calc(100vh - 56px);
}

.image-side {
  width: 600px;
  height: 600px;
  margin: auto 0;
  margin-left: 2rem;
}

.auth-image {
  width: 100%;
  height: 100%;
  object-fit: cover;
  border-radius: 8px;
}

.form-side {
  flex: 1;
  padding: 2rem;
  display: flex;
  align-items: center;
  justify-content: center;
}

.card {
  background: transparent;
  width: 100%;
  max-width: 600px;
}
</style>
