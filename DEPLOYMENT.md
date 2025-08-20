# Deployment Instructions

## Step 1: Push Buildpack to GitHub

1. Create a new repository on GitHub (e.g., `heroku-buildpack-imagemagick-fonts`)
2. Push this buildpack code to that repository:

```bash
cd heroku-buildpack-imagemagick-fonts
git init
git add .
git commit -m "Initial commit: ImageMagick buildpack with font support"
git branch -M main
git remote add origin https://github.com/[your-username]/heroku-buildpack-imagemagick-fonts.git
git push -u origin main
```

## Step 2: Add Buildpack to Your Heroku App

```bash
# Remove old ImageMagick buildpack if you have one
heroku buildpacks:clear --app YOUR_APP_NAME

# Add the new buildpack (replace with your GitHub repo URL)
heroku buildpacks:add https://github.com/[your-username]/heroku-buildpack-imagemagick-fonts --index 1 --app YOUR_APP_NAME

# Add your Ruby buildpack
heroku buildpacks:add heroku/ruby --app YOUR_APP_NAME

# Verify buildpacks are set correctly
heroku buildpacks --app YOUR_APP_NAME
```

## Step 3: Deploy

```bash
# Clear cache to ensure fresh build
heroku plugins:install heroku-repo
heroku repo:purge_cache --app YOUR_APP_NAME

# Deploy
git push heroku main
```

## Step 4: Test

After deployment, test the watermarking service:

```bash
heroku run rails console --app YOUR_APP_NAME

# In the Rails console:
ImageWatermarkService.add_watermark("https://oaidalleapiprodscus.blob.core.windows.net/private/org-lgjy6EPMRd7f0XGCxTZONAJq/user-0wSUkhzUUy7efHUbi3R9lNv8/img-0jLlM7YGUDld7wE18bQAJcq0.png?st=2025-08-20T17%3A38%3A00Z&se=2025-08-20T19%3A38%3A00Z&sp=r&sv=2024-08-04&sr=b&rscd=inline&rsct=image/png&skoid=77e5a8ec-6bd1-4477-8afc-16703a64f029&sktid=a48cca56-e6da-484e-a814-9c849652bcb3&skt=2025-08-19T22%3A57%3A33Z&ske=2025-08-20T22%3A57%3A33Z&sks=b&skv=2024-08-04&sig=JVlbezTtmEks0rXiIinlUrwKgq9XfIrW81NOU18EUnk%3D")
```

This should now work without font errors!

## Troubleshooting

If you still get font errors:

1. Check that the buildpack is installed first: `heroku buildpacks --app YOUR_APP_NAME`
2. Clear cache and redeploy: `heroku repo:purge_cache --app YOUR_APP_NAME`
3. Check build logs: `heroku logs --tail --app YOUR_APP_NAME`
4. Test font availability: `heroku run "magick -list font" --app YOUR_APP_NAME`

## Key Improvements Over Old Buildpack

- ✅ **Latest ImageMagick**: 7.1.1-36 vs 7.0.5-10 (2017)
- ✅ **Font Support**: Full DejaVu font family included
- ✅ **Font Configuration**: Complete type.xml and fonts.conf
- ✅ **Security**: Modern policy.xml with resource limits
- ✅ **Error Handling**: Robust build process with fallbacks
- ✅ **Documentation**: Comprehensive README and troubleshooting guide