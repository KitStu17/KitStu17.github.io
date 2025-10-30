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

.experience-container,
.skills-container {
  display: flex;
  gap: 3rem;
  margin: 2rem 0;
  padding: 2rem 0;
}

.experience-left,
.skills-left {
  flex: 0 0 250px;
  padding: 1.5rem;
  border-radius: 8px;
}

.experience-left h3,
.skills-left h3 {
  margin-top: 0;
  font-size: 1.3rem;
  margin-bottom: 0.5rem;
}

.experience-left a,
.skills-left a {
  color: #1976d2;
  font-size: 0.9rem;
  text-decoration: none;
}

.experience-left p,
.skills-left p {
  color: #666;
  font-size: 0.9rem;
  margin: 0.5rem 0;
}

.experience-right,
.skills-right {
  flex: 1;
  padding-top: 0.5rem;
}

.experience-right ul,
.skills-right ul {
  list-style: disc;
  padding-left: 1.5rem;
}

.experience-right li,
.skills-right li {
  margin-bottom: 0.8rem;
  line-height: 1.6;
}

@media (max-width: 768px) {
  .experience-container,
  .skills-container {
    flex-direction: column;
  }

  .experience-left,
  .skills-left {
    flex: 1;
  }
}
</style>

# PORTFOLIO

<div class="navigator">
  <h3>Navigator</h3>
  <ol>
    <li><a href="#about-me">About Me</a></li>
    <li><a href="#experience">Experience</a></li>
    <li><a href="#skills">Skills</a></li>
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

## Experience

<div class="experience-container">
  <div class="experience-left">
    <h3>(주)제이니스</h3>
    <a href="http://www.jness.co.kr/" target="_blank">http://www.jness.co.kr/</a>
    <p>웹 개발자</p>
    <p>2024.06.17 ~ 2025.09.30</p>
  </div>
  <div class="experience-right">
    <p style="font-style: italic; color: #666; margin-bottom: 1.5rem;">PC-OFF 시장 점유율 1위의 중소기업</p>
    <ul>
      <li>PostgreSQL 성능 튜닝 및 장애 대응</li>
      <li>레거시 Spring 프로젝트를 Spring Boot 기반으로 전환</li>
      <li>정기결제 스케줄링 시스템 개선</li>
      <li>신규 서비스 PineWorks의 구독 관리, 결제, 환불 등 핵심 비즈니스 로직 개선</li>
    </ul>
  </div>
</div>

---

## Skills

### 레거시 프로젝트 리펙토링
- MOffice 프로젝트 마이그레이션(Legacy Spring Framework -> Spring Boot)
- 사내 통합 포탈 사이트 마이그레이션(JSP -> Spring Cloud, Spring WebFlux)

### 대용량 데이터 처리 및 성능 개선
- PineWorks 프로젝트 내 PostgreSQL SlowQuery 튜닝
- 계약사 100건 이상의 사용자 개인정보 접근 로그 시스템 개발
- JDBC/PostgreSQL 성능 개선

### 보안 중심 아키텍처 설계
- 보안 취약점 진단 및 대응을 통한 코드 레벨 보안 강화
- Spring Security SSO 통합 인증 시스템 개발
- Spring Cloud Gateway 기반 API 라우팅 구조 설계

---

## Stacks

<div class="stacks-grid">
  <div>
    <h3>Back-End</h3>
    <ul>
      <li>Java / Spring</li>
      <li>Spring boot</li>
      <li>Spring MVC / WebFlux</li>
      <li>NodeJS</li>
      <li>Spring JDBC</li>
      <li>Mybatis</li>
      <li>Gradle</li>
    </ul>
  </div>

  <div>
    <h3>Tools</h3>
    <ul>
      <li>IntelliJ</li>
      <li>VSCode</li>
      <li>Eclipse</li>
    </ul>
  </div>

  <div>
    <h3>DB/Cache</h3>
    <ul>
      <li>PostgreSQL</li>
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

<div class="projects-grid">
  <div class="project-card" onclick="location.href='{{ site.baseurl }}/posts/BUSS-프로젝트-정리/'" style="cursor: pointer;">
    <img src="/assets/img/buss/project_main.png" alt="BUSS" class="project-thumbnail">
    <div class="project-content">
      <h3>[BUSS] 버스 정보 안내 및 활용방법 제시 어플리케이션</h3>
      <span class="project-tag">팀 프로젝트</span>
      <p>2022년 10월 3일 → 2022년 12월 5일</p>
    </div>
  </div>

  <div class="project-card" onclick="location.href='{{ site.baseurl }}/posts/ABTwin-프로젝트-정리/'" style="cursor: pointer;">
    <img src="/assets/img/abtwin/project_main.png" alt="ABTwin" class="project-thumbnail">
    <div class="project-content">
      <h3>[ABTwin] 밸런스게임 어플리케이션</h3>
      <span class="project-tag">팀 프로젝트</span>
      <p>2022년 3월 2일 → 2023년 6월 17일</p>
    </div>
  </div>

  <div class="project-card" onclick="location.href='{{ site.baseurl }}/posts/Toys-프로젝트-정리/'" style="cursor: pointer;">
    <img src="/assets/img/toys/project_main.png" alt="Toys" class="project-thumbnail">
    <div class="project-content">
      <h3>[Toys] 토이 프로젝트 인원 모집 사이트</h3>
      <span class="project-tag">개인 프로젝트</span>
      <p>2023년 10월 10일 → 2023년 10월 23일</p>
    </div>
  </div>

  <div class="project-card" onclick="location.href='{{ site.baseurl }}/posts/플래시-프로젝트-정리/'" style="cursor: pointer;">
    <img src="/assets/img/flash/project_main.png" alt="플래시" class="project-thumbnail">
    <div class="project-content">
      <h3>[플래시] 지자체 복지 안내 플랫폼</h3>
      <span class="project-tag">팀 프로젝트</span>
      <p>2023년 10월 25일 → 2023년 11월 27일</p>
    </div>
  </div>
</div>
