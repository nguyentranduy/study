# Xây dựng Portfolio

Portfolio là bộ sưu tập các dự án và công việc bạn đã làm, giúp showcase kỹ năng thực tế của bạn.

## Tại sao cần Portfolio?

- **Chứng minh năng lực** - CV nói bạn biết gì, Portfolio chứng minh bạn làm được gì
- **Nổi bật** - Đặc biệt quan trọng với fresher khi chưa có nhiều kinh nghiệm
- **Thể hiện đam mê** - Cho thấy bạn code ngoài giờ làm việc
- **Talking points** - Có gì để nói trong phỏng vấn

---

## Các thành phần của Portfolio

### 1. GitHub Profile

GitHub là portfolio quan trọng nhất của developer.

#### Profile README

Tạo file `README.md` trong repo có tên trùng với username.

```markdown
# Hi, I'm Nguyen Van A 👋

## About Me
Java Developer with 3 years of experience in building web applications.
Passionate about clean code and continuous learning.

## Tech Stack
![Java](https://img.shields.io/badge/Java-ED8B00?style=flat&logo=java&logoColor=white)
![Spring](https://img.shields.io/badge/Spring-6DB33F?style=flat&logo=spring&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-316192?style=flat&logo=postgresql&logoColor=white)

## GitHub Stats
![GitHub Stats](https://github-readme-stats.vercel.app/api?username=nguyenvana&show_icons=true)

## Featured Projects
- [E-Commerce Platform](https://github.com/nguyenvana/ecommerce) - Full-stack e-commerce with Spring Boot
- [Task Manager API](https://github.com/nguyenvana/task-api) - RESTful API with JWT authentication

## Connect with Me
[![LinkedIn](https://img.shields.io/badge/LinkedIn-0077B5?style=flat&logo=linkedin)](https://linkedin.com/in/nguyenvana)
[![Email](https://img.shields.io/badge/Email-D14836?style=flat&logo=gmail)](mailto:nguyenvana@email.com)
```

#### Pinned Repositories

Pin 6 repos tốt nhất lên profile:

- 2-3 dự án cá nhân hoàn chỉnh
- 1-2 contributions vào open source
- 1 repo về learning/notes (optional)

#### Repository README

Mỗi project cần README chi tiết:

```markdown
# E-Commerce Platform

A full-stack e-commerce application built with Spring Boot and React.

## Demo
🔗 [Live Demo](https://ecommerce-demo.com)

## Screenshots
![Homepage](screenshots/home.png)
![Product Page](screenshots/product.png)

## Features
- User authentication with JWT
- Product catalog with search and filters
- Shopping cart and checkout
- Payment integration with VNPay
- Admin dashboard

## Tech Stack
**Backend:** Java 17, Spring Boot 3, PostgreSQL, Redis
**Frontend:** React, TypeScript, Tailwind CSS
**DevOps:** Docker, GitHub Actions, AWS

## Getting Started

### Prerequisites
- Java 17+
- Node.js 18+
- PostgreSQL 14+

### Installation
```bash
# Clone the repo
git clone https://github.com/nguyenvana/ecommerce.git

# Backend
cd backend
./mvnw spring-boot:run

# Frontend
cd frontend
npm install && npm start
```

## API Documentation
API docs available at `/swagger-ui.html` when running locally.

## Architecture
```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│   React     │────▶│ Spring Boot │────▶│ PostgreSQL  │
│  Frontend   │     │   Backend   │     │  Database   │
└─────────────┘     └─────────────┘     └─────────────┘
                           │
                           ▼
                    ┌─────────────┐
                    │    Redis    │
                    │    Cache    │
                    └─────────────┘
```

## Contributing
Pull requests are welcome. For major changes, please open an issue first.

## License
MIT
```

---

### 2. Personal Website

Website cá nhân giúp bạn kiểm soát hoàn toàn cách thể hiện bản thân.

#### Các trang cần có

```
/                   - Homepage (giới thiệu ngắn)
/about              - About me (chi tiết hơn)
/projects           - Danh sách projects
/projects/:id       - Chi tiết từng project
/blog               - Blog posts (optional)
/contact            - Thông tin liên hệ
```

#### Tech stack đề xuất

| Option | Pros | Cons |
|--------|------|------|
| **GitHub Pages + Jekyll** | Free, simple | Limited customization |
| **Vercel + Next.js** | Fast, modern, free | Learning curve |
| **Netlify + Gatsby** | Great for blogs | Build time |
| **Custom domain** | Professional | Cost ~$10/year |

#### Ví dụ cấu trúc

```
portfolio/
├── public/
│   ├── images/
│   └── resume.pdf
├── src/
│   ├── components/
│   │   ├── Header.jsx
│   │   ├── ProjectCard.jsx
│   │   └── Footer.jsx
│   ├── pages/
│   │   ├── index.jsx
│   │   ├── about.jsx
│   │   ├── projects.jsx
│   │   └── contact.jsx
│   └── data/
│       └── projects.json
└── package.json
```

---

### 3. LinkedIn Profile

LinkedIn là nơi recruiters tìm kiếm candidates.

#### Optimize Profile

- **Photo** - Professional headshot
- **Headline** - Không chỉ job title, thêm value proposition
  ```
  ❌ "Software Developer at ABC Company"
  ✅ "Java Developer | Spring Boot | Building scalable web applications"
  ```
- **About** - 3-5 paragraphs về bạn, skills, và goals
- **Experience** - Chi tiết như CV, có thể dài hơn
- **Skills** - Thêm đủ skills, xin endorsements
- **Recommendations** - Xin từ đồng nghiệp, managers

#### Activity

- Share technical articles
- Comment on industry posts
- Write posts về learning journey
- Engage với community

---

## Chọn Projects cho Portfolio

### Tiêu chí chọn project

1. **Hoàn chỉnh** - Có thể demo được, không phải WIP
2. **Relevant** - Liên quan đến job bạn muốn apply
3. **Showcase skills** - Thể hiện technical skills
4. **Có depth** - Không quá đơn giản (todo app)

### Project ideas theo level

#### Fresher

| Project | Skills thể hiện |
|---------|-----------------|
| Personal Blog | CRUD, Authentication, Database |
| Task Manager | REST API, Frontend integration |
| Weather App | API consumption, UI/UX |
| URL Shortener | Database design, Caching |

#### Junior

| Project | Skills thể hiện |
|---------|-----------------|
| E-commerce | Full-stack, Payment, Complex logic |
| Chat Application | WebSocket, Real-time |
| Job Board | Search, Filtering, Pagination |
| Booking System | Date handling, Transactions |

#### Senior

| Project | Skills thể hiện |
|---------|-----------------|
| Microservices Demo | Architecture, Docker, K8s |
| Open Source Contribution | Collaboration, Code quality |
| Technical Blog | Communication, Expertise |
| CLI Tool/Library | API design, Documentation |

---

## Project Documentation

### Mỗi project cần có

1. **README.md** - Overview, setup, usage
2. **Screenshots/Demo** - Visual proof
3. **Architecture diagram** - System design
4. **API documentation** - Nếu có backend
5. **Deployment** - Live demo nếu có thể

### Ví dụ structure

```
project/
├── README.md           # Main documentation
├── docs/
│   ├── ARCHITECTURE.md # System design
│   ├── API.md          # API documentation
│   └── DEPLOYMENT.md   # How to deploy
├── screenshots/
│   ├── home.png
│   ├── dashboard.png
│   └── demo.gif
└── src/
    └── ...
```

---

## Tips tạo Portfolio ấn tượng

### Code Quality

```java
// ❌ Bad: No comments, unclear naming
public void p(List<Integer> l) {
    for(int i=0;i<l.size();i++) {
        System.out.println(l.get(i));
    }
}

// ✅ Good: Clean, readable
public void printNumbers(List<Integer> numbers) {
    numbers.forEach(System.out::println);
}
```

### Commit Messages

```bash
# ❌ Bad
git commit -m "fix"
git commit -m "update"
git commit -m "asdfasdf"

# ✅ Good
git commit -m "feat: add user authentication with JWT"
git commit -m "fix: resolve null pointer in OrderService"
git commit -m "docs: update API documentation"
```

### Consistent Activity

- Commit regularly (không cần daily, nhưng consistent)
- Avoid "commit bombing" (nhiều commits cùng ngày)
- Show progression over time

---

## Checklist Portfolio

### GitHub

- [ ] Profile README hoàn chỉnh
- [ ] 4-6 pinned repositories
- [ ] Mỗi repo có README chi tiết
- [ ] Code clean, có comments khi cần
- [ ] Commit messages rõ ràng
- [ ] Có contributions graph xanh

### Projects

- [ ] Ít nhất 2-3 projects hoàn chỉnh
- [ ] Có live demo hoặc screenshots
- [ ] Documentation đầy đủ
- [ ] Relevant với job target

### Online Presence

- [ ] LinkedIn profile optimized
- [ ] Personal website (optional but recommended)
- [ ] Consistent branding across platforms

---

## Tài nguyên

### Inspiration

- [GitHub Profile README Examples](https://github.com/abhisheknaiidu/awesome-github-profile-readme)
- [Developer Portfolio Examples](https://github.com/emmabostian/developer-portfolios)

### Tools

- [GitHub Profile README Generator](https://rahuldkjain.github.io/gh-profile-readme-generator/)
- [Shields.io](https://shields.io/) - Badges cho README
- [Carbon](https://carbon.now.sh/) - Code screenshots

## Tiếp theo

- [Chuẩn bị phỏng vấn](../interview/chuan-bi.md)
