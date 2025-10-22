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

.project-card {
  border: 1px solid #e0e0e0;
  border-radius: 8px;
  padding: 1.5rem;
  margin-bottom: 1.5rem;
  transition: box-shadow 0.3s;
}

.project-card:hover {
  box-shadow: 0 4px 12px rgba(0,0,0,0.1);
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

## About Me

<div class="profile-section">
  <div class="profile-left">
    <img src="/assets/img/profile.jpg" alt="Profile" class="profile-img">
    <h3>Park Jonghyun</h3>
    <p style="text-align: left; font-size: 0.9rem; line-height: 1.6;">
      Wonju, South Korea<br>
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

<div class="navigator">
  <h3>Navigator</h3>
  <ol>
    <li><a href="#about-me">About Me</a></li>
    <li><a href="#stacks">Stacks</a></li>
    <li><a href="#projects">Projects</a></li>
  </ol>
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

<a href="{% post_url 2024-02-15-BUSS 프로젝트 정리 %}" style="text-decoration: none; color: inherit;">
  <div class="project-card">
    <h3>[BUSS] 버스 정보 안내 및 활용방법 제시 어플리케이션</h3>
    <p><strong>작업기간:</strong> 2022년 10월 3일 → 2022년 12월 5일</p>
    <p><strong>기여도:</strong> 40%</p>
    <span class="project-tag">팀 프로젝트</span>
  </div>
</a>

<a href="{% post_url 2024-02-14-ABTwin 프로젝트 정리 %}" style="text-decoration: none; color: inherit;">
  <div class="project-card">
    <h3>[ABTwin] 밸런스게임 어플리케이션</h3>
    <p><strong>작업기간:</strong> 2022년 3월 2일 → 2023년 6월 17일</p>
    <p><strong>기여도:</strong> 40%</p>
    <span class="project-tag">팀 프로젝트</span>
  </div>
</a>

<a href="{% post_url 2024-02-15-Toys 프로젝트 정리 %}" style="text-decoration: none; color: inherit;">
  <div class="project-card">
    <h3>[Toys] 토이 프로젝트 인원 모집 사이트</h3>
    <p><strong>작업기간:</strong> 2023년 10월 10일 → 2023년 10월 23일</p>
    <p><strong>기여도:</strong> 100%</p>
    <span class="project-tag">개인 프로젝트</span>
  </div>
</a>

<a href="{% post_url 2024-02-16-플래시 프로젝트 정리 %}" style="text-decoration: none; color: inherit;">
  <div class="project-card">
    <h3>[플래시] 지자체 복지 안내 플랫폼</h3>
    <p><strong>작업기간:</strong> 2023년 10월 25일 → 2023년 11월 27일</p>
    <p><strong>기여도:</strong> 25%</p>
    <span class="project-tag">팀 프로젝트</span>
  </div>
</a>
