---
# the default layout is 'page'
icon: fas fa-info-circle
order: 4
---

<style>
.profile-section {
  display: flex;
  gap: 3rem;
  margin-bottom: 3rem;
}

.profile-left {
  flex: 0 0 250px;
  text-align: center;
}

.profile-img {
  width: 200px;
  border-radius: 8px;
  margin-bottom: 1rem;
}

.profile-right {
  flex: 1;
}

.navigator {
  background-color: #f8f9fa;
  padding: 1.5rem;
  border-radius: 8px;
  margin: 2rem 0;
}

.navigator h3 {
  margin-top: 0;
  margin-bottom: 1rem;
}

.navigator ol {
  margin: 0;
  padding-left: 1.5rem;
}

.stacks-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: 2rem;
  margin: 2rem 0;
}

.projects-grid {
  display: flex;
  flex-direction: column;
  gap: 1rem;
  margin: 2rem 0;
}

.project-card {
  border: 1px solid #e0e0e0;
  border-radius: 12px;
  padding: 1.5rem;
  transition: all 0.3s;
  display: flex;
  align-items: center;
  gap: 1.5rem;
  background-color: #f8f9fa;
}

.project-card:hover {
  box-shadow: 0 4px 12px rgba(0,0,0,0.1);
  transform: translateY(-2px);
}

.project-thumbnail {
  width: 120px;
  height: 120px;
  object-fit: cover;
  border-radius: 8px;
  flex-shrink: 0;
}

.project-content {
  flex: 1;
  min-width: 0;
}

.project-content h3 {
  font-size: 1.1rem;
  margin: 0 0 0.5rem 0;
  word-break: keep-all;
  word-wrap: break-word;
}

.project-content p {
  font-size: 0.85rem;
  color: #666;
  margin: 0.5rem 0;
  word-break: keep-all;
  word-wrap: break-word;
}

@media (max-width: 768px) {
  .project-card {
    flex-direction: column;
    align-items: flex-start;
  }

  .project-thumbnail {
    width: 100%;
    height: 150px;
  }
}

.project-tag {
  display: inline-block;
  background-color: #e3f2fd;
  color: #1976d2;
  padding: 0.25rem 0.75rem;
  border-radius: 4px;
  font-size: 0.875rem;
  margin-top: 0.5rem;
}
</style>

# PORTFOLIO

<div class="navigator">
  <h3>Navigator</h3>
  <ol>
    <li><a href="#about-me">About Me</a></li>
    <li><a href="#stacks">Stacks</a></li>
    <li><a href="#projects">Projects</a></li>
  </ol>
</div>

## About Me

<div class="profile-section">
  <div class="profile-left">
    <img src="/assets/img/profile.jpg" alt="Profile" class="profile-img">
    <h3>Park Jonghyun</h3>
    <p style="text-align: left; font-size: 0.9rem; line-height: 1.6;">
      Seoul, Myeonmok-dong, South Korea<br>
      Tel. 010-5053-5474<br>
      E. kiri6077@naver.com
    </p>
  </div>

  <div class="profile-right">
    <h3 style="color: #1976d2;">Introduce</h3>
    <p>주니어 개발자 박종현입니다.</p>
    <p>Spring Boot를 사용한 백엔드 개발을 하고 있습니다.</p>
    <p>GitHub 링크: <a href="https://github.com/KitStu17">https://github.com/KitStu17</a></p>

    <h3 style="color: #1976d2; margin-top: 2rem;">Certificate</h3>
    <ul>
      <li>정보처리기사</li>
      <li>SQLD</li>
    </ul>

    <h3 style="color: #1976d2; margin-top: 2rem;">Career</h3>
    <ul>
      <li>(주) 제이니스 기업부설연구소 - 박종현 연구원 (1년 4개월)</li>
    </ul>

    <h3 style="color: #1976d2; margin-top: 2rem;">Education</h3>
    <ul>
      <li>금오공과대학교 컴퓨터공학과 2017.03 - 2024.02</li>
      <li>Webkit640 3기 수료 2023.07-2023.11</li>
    </ul>

    <h3 style="color: #1976d2; margin-top: 2rem;">Awards</h3>
    <ul>
      <li>한국정보기술학회 대학생 논문경진대회(동상) 2023.06.02</li>
      <li>웹킷640 3기 팀프로젝트 경연(우수상) 2023.11.27</li>
      <li>SW전문인재양성 컨소시엄(장려상) 2023.12.06</li>
    </ul>
  </div>
</div>

---

## Stacks

<div class="stacks-grid">
  <div>
    <h3>Back-End</h3>
    <ul>
      <li>Java / Spring</li>
      <li>Spring boot</li>
      <li>NodeJS</li>
      <li>Spring JPA</li>
      <li>Mybatis</li>
    </ul>
  </div>

  <div>
    <h3>Tools</h3>
    <ul>
      <li>IntelliJ</li>
      <li>VSCode</li>
      <li>Eclipse</li>
      <li>Android Studio</li>
    </ul>
  </div>

  <div>
    <h3>DB/Cache</h3>
    <ul>
      <li>MySQL</li>
      <li>Maria DB</li>
      <li>Firebase</li>
      <li>Mongo DB</li>
    </ul>
  </div>

  <div>
    <h3>Collaborations</h3>
    <ul>
      <li>Github</li>
      <li>Discord</li>
    </ul>
  </div>

  <div>
    <h3>Server</h3>
    <ul>
      <li>AWS EC2</li>
    </ul>
  </div>
</div>

---

## Projects

{% linkpreview "{{ site.url }}/posts/BUSS-프로젝트-정리/" %}

{% linkpreview "{{ site.url }}/posts/ABTwin-프로젝트-정리/" %}

{% linkpreview "{{ site.url }}/posts/Toys-프로젝트-정리/" %}

{% linkpreview "{{ site.url }}/posts/플래시-프로젝트-정리/" %}
