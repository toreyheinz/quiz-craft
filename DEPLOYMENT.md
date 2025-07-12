# Cloudflare Pages Deployment Guide

## Prerequisites
- Cloudflare account
- GitHub repository with this code

## Deployment Steps

1. **Build the project locally** (optional, to test):
   ```bash
   npm install
   npm run build
   ```

2. **Push to GitHub**:
   ```bash
   git init
   git add .
   git commit -m "Initial commit"
   git remote add origin YOUR_GITHUB_REPO_URL
   git push -u origin main
   ```

3. **Deploy to Cloudflare Pages**:
   - Go to [Cloudflare Pages](https://pages.cloudflare.com/)
   - Click "Create a project"
   - Connect to Git and select your repository
   - Configure build settings:
     - Build command: `npm run build`
     - Build output directory: `dist`
     - Root directory: `/assessment-app` (if in subdirectory)
   - Click "Save and Deploy"

4. **Custom Domain** (optional):
   - After deployment, go to Custom domains
   - Add your domain (e.g., `assessment-app.example.com`)
   - Update DNS records as instructed

## Adding New Assessments

1. Create a new JSON file in `public/assessments/` (e.g., `11th-grade-assessment.json`)
2. Follow the same structure as existing assessment files
3. Add the assessment to the home page in `src/views/Home.vue`
4. Commit and push - Cloudflare will automatically rebuild

## Local Development

```bash
npm install
npm run dev
```

Visit http://localhost:5173 to see the app.