<template>
  <div class="about-container">
    <div class="content-wrapper">
      <h1 class="text-center mb-5">About Recipe App</h1>
      
      <div class="row g-4">
        <div class="col-md-6">
          <div class="card h-100">
            <div class="card-body">
              <h2 class="card-title mb-4">
                <i class="fas fa-code me-2"></i>Development Team
              </h2>
              <div class="team-members">
                <div class="member mb-3">
                  <h5>Ofek Avigezer</h5>
                  <p>ID: 325816569</p>
                </div>
                <div class="member mb-3">
                  <h5>Sean Shuhman</h5>
                  <p>ID: 325944635</p>
                </div>
              </div>
            </div>
          </div>
        </div>

        <div class="col-md-6">
          <div class="card h-100">
            <div class="card-body">
              <h2 class="card-title mb-4">
                <i class="fas fa-bullhorn me-0.5"></i>Our Mission
              </h2>
              <p class="marketing-text">
                Welcome to RecipeApp - your ultimate culinary companion! We're revolutionizing 
                the way home cooks discover, create, and share recipes. Our platform combines 
                the warmth of traditional family recipes with modern recipe management technology. 
                Whether you're a seasoned chef or just starting your cooking journey, RecipeApp 
                provides an intuitive, user-friendly experience to explore global cuisines, 
                save your favorites, and build your personal recipe collection.
              </p>
              <p class="marketing-text">
                Built with cutting-edge technology and designed with passion, RecipeApp brings 
                the joy of cooking to your fingertips. Join our growing community of food 
                enthusiasts and transform your cooking experience today!
              </p>
            </div>
          </div>
        </div>

        <div class="col-md-6">
          <div class="card h-100">
            <div class="card-body">
              <h2 class="card-title mb-4">
                <i class="fas fa-users me-0.5"></i>About the Developers
              </h2>
              <div class="developers-info">
                <div class="developer-section mb-3">
                  <h5><i class="fas fa-user-graduate me-0.5"></i>Academic Background</h5>
                  <p>We are third-year students at Ben-Gurion University, studying Software and Information Systems Engineering.</p>
                </div>
                <div class="developer-section mb-3">
                  <h5><i class="fas fa-code me-0.5"></i>Technical Expertise</h5>
                  <p>Specializing in web development, with experience in both frontend and backend technologies. Our focus is on creating user-friendly and efficient web applications.</p>
                </div>
                <div class="developer-section">
                  <h5><i class="fas fa-laptop-code me-0.5"></i>Project Role</h5>
                  <p>Collaborated on all aspects of the RecipeApp, from design and implementation to testing and deployment, ensuring a high-quality user experience.</p>
                </div>
              </div>
            </div>
          </div>
        </div>

        <div class="col-md-6">
          <div class="card h-100">
            <div class="card-body">
              <h2 class="card-title mb-4">
                <i class="fas fa-tasks me-0.5"></i>Project Overview
              </h2>
              <p>A recipe management system built with Vue.js and Node.js, allowing users to:</p>
              <ul class="project-features">
                <li>Browse and search recipes</li>
                <li>Create and manage personal recipes</li>
                <li>Save favorite recipes</li>
                <li>Access family recipes</li>
                <li>Get personalized recipe recommendations</li>
              </ul>
            </div>
          </div>
        </div>

        <div class="col-12">
          <div class="card">
            <div class="card-body">
              <h2 class="card-title mb-4">
                <i class="fas fa-history me-2"></i>Previous Assignments
              </h2>
              <div class="assignments-grid">
                <a href="https://wed-2023.github.io/325816569/" 
                   class="assignment-link" target="_blank">
                  <i class="fab fa-github me-2"></i>Assignment 1 Ofek
                </a>
                <a href="https://wed-2023.github.io/325944635/#about" 
                   class="assignment-link" target="_blank">
                  <i class="fab fa-github me-2"></i>Assignment 1 Sean
                </a>
                <a href="https://wed-2023.github.io/assignment2-325816569_325944635_assignment2/" 
                   class="assignment-link" target="_blank">
                  <i class="fab fa-github me-2"></i>Assignment 2 
                </a>
                <a href="https://github.com/WED-2023/assignment3-2-325816569_325944635_assignment2" 
                   class="assignment-link" target="_blank">
                  <i class="fab fa-github me-2"></i>Assignment 3.2 Repository
                </a>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.about-container {
  min-height: calc(100vh - 56px);
  background-color: #e6f4ea;
  padding: 2rem 0;
}

.content-wrapper {
  max-width: 1200px;
  margin: 0 auto;
  padding: 0 1rem;
}

.card {
  background-color: white;
  border: none;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
  transition: transform 0.2s;
}

.card:hover {
  transform: translateY(-5px);
}

.card-title {
  color: #4caf50;
  font-size: 1.5rem;
}

.project-features {
  list-style: none;
  padding-left: 0;
}

.project-features li {
  padding: 0.5rem 0;
  border-bottom: 1px solid #e6f4ea;
}

.project-features li:before {
  content: "•";
  color: #4caf50;
  font-weight: bold;
  margin-right: 0.5rem;
}

.assignments-grid {
  display: grid;
  gap: 1rem;
}

.assignment-link {
  display: inline-block;
  padding: 0.75rem 1rem;
  background-color: #e6f4ea;
  color: #4caf50;
  text-decoration: none;
  border-radius: 4px;
  transition: all 0.3s ease;
}

.assignment-link:hover {
  background-color: #4caf50;
  color: white;
}

.team-members .member {
  padding: 1rem;
  background-color: #e6f4ea;
  border-radius: 4px;
  margin-bottom: 1rem;
}

.team-members .member h5 {
  color: #4caf50;
  margin-bottom: 0.5rem;
}

.team-members .member p {
  margin-bottom: 0;
  color: #666;
}

.marketing-text {
  line-height: 1.6;
  color: #666;
  margin-bottom: 1rem;
}

.marketing-text:last-child {
  margin-bottom: 0;
}

.developers-info {
  color: #666;
}

.developer-section h5 {
  color: #4caf50;
  margin-bottom: 0.75rem;
  font-size: 1.1rem;
}

.developer-section p {
  line-height: 1.6;
  margin-bottom: 1rem;
}
</style>
