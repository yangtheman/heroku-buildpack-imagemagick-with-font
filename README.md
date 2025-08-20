# Heroku Buildpack: ImageMagick with Fonts

A modern Heroku buildpack for installing ImageMagick with full font support, specifically designed to fix common font-related errors in containerized environments.

## Features

- **Latest ImageMagick**: Uses ImageMagick 7.1.1-36 (latest stable as of 2025)
- **Font Support**: Includes DejaVu font family with proper font configuration
- **Security Focused**: Includes secure policy configuration with resource limits
- **Optimized for Heroku**: Configured specifically for Heroku's ephemeral filesystem
- **Caching**: Smart caching to reduce build times on subsequent deploys
- **Error Handling**: Robust error handling and fallback mechanisms

## Common Issues This Solves

- `UnableToOpenConfigureFile 'type.xml'`
- `UnableToReadFont 'Arial'` 
- `UnableToReadFont` errors in general
- Font rendering failures in ImageMagick operations
- Missing font configuration in containerized environments

## Installation

Add this buildpack to your Heroku application:

```bash
heroku buildpacks:add https://github.com/[your-username]/heroku-buildpack-imagemagick-fonts --index 1 --app YOUR_APP_NAME
```

**Important**: Use `--index 1` to ensure ImageMagick is installed before other buildpacks that might depend on it.

## Usage

After installation, ImageMagick will be available with the following fonts:

- **DejaVu Sans** (`DejaVu-Sans`)
- **DejaVu Sans Bold** (`DejaVu-Sans-Bold`) 
- **DejaVu Sans Oblique** (`DejaVu-Sans-Oblique`)
- **DejaVu Serif** (`DejaVu-Serif`)
- **DejaVu Sans Mono** (`DejaVu-Sans-Mono`)

### Ruby/Rails Example

```ruby
require 'mini_magick'

image = MiniMagick::Image.open('input.jpg')
image.combine_options do |c|
  c.font 'DejaVu-Sans'  # This will now work!
  c.pointsize 40
  c.fill 'white'
  c.gravity 'center'
  c.annotate '+0+0', 'Hello World'
end
image.write 'output.jpg'
```

### Environment Variables

The buildpack sets up the following environment variables:

- `PATH`: Includes ImageMagick binaries
- `LD_LIBRARY_PATH`: Includes ImageMagick libraries
- `MAGICK_CONFIGURE_PATH`: Points to ImageMagick configuration
- `FONTCONFIG_PATH`: Points to font configuration
- `FONTCONFIG_FILE`: Points to fonts.conf file
- `MAGICK_FONT_PATH`: Points to installed fonts
- `MAGICK_HOME`: Points to ImageMagick installation directory

## Configuration

### Custom ImageMagick Version

Set the `IMAGE_MAGICK_VERSION` environment variable:

```bash
heroku config:set IMAGE_MAGICK_VERSION=7.1.1-30 --app YOUR_APP_NAME
```

### Security Policy

The buildpack includes a secure policy configuration that:
- Disables dangerous coders (EPHEMERAL, HTTPS, MVG, MSL, etc.)
- Sets resource limits (256MB memory, 1GB disk, 120s timeout)
- Restricts image dimensions (16KP width/height, 128MP area)

## Troubleshooting

### Font Issues

If you're still getting font errors:

1. **Check available fonts**:
   ```bash
   heroku run "magick -list font" --app YOUR_APP_NAME
   ```

2. **Use the exact font names** from the type.xml configuration:
   - `DejaVu-Sans` (not `DejaVu Sans`)
   - `DejaVu-Sans-Bold` 
   - `DejaVu-Serif`
   - `DejaVu-Sans-Mono`

3. **Fallback strategy**: Don't specify a font to use system defaults:
   ```ruby
   image.combine_options do |c|
     # c.font 'DejaVu-Sans'  # Comment out font specification
     c.pointsize 40
     c.annotate '+0+0', 'Text'
   end
   ```

### Build Issues

1. **Clear cache** if you're having build problems:
   ```bash
   heroku plugins:install heroku-repo
   heroku repo:purge_cache --app YOUR_APP_NAME
   ```

2. **Check build logs** for specific error messages:
   ```bash
   heroku logs --tail --app YOUR_APP_NAME
   ```

### Memory Issues

The buildpack sets conservative resource limits. If you need to process larger images:

1. **Increase Heroku dyno size** (more RAM)
2. **Process images in smaller batches**
3. **Consider using a background job processor**

## Technical Details

### Installation Process

1. Downloads ImageMagick source code from GitHub
2. Compiles with optimizations for Heroku environment
3. Downloads and installs DejaVu fonts
4. Creates font configuration files
5. Sets up ImageMagick policy and type configuration
6. Caches the complete installation for faster subsequent builds

### File Locations

- **ImageMagick**: `$HOME/vendor/imagemagick/`
- **Fonts**: `$HOME/vendor/imagemagick/share/fonts/truetype/dejavu/`
- **Configuration**: `$HOME/vendor/imagemagick/etc/ImageMagick-7/`
- **Font Config**: `$HOME/vendor/imagemagick/etc/fonts/`

## Comparison with Other Buildpacks

| Feature | This Buildpack | DuckyTeam/heroku-buildpack-imagemagick |
|---------|---------------|----------------------------------------|
| ImageMagick Version | 7.1.1-36 (2025) | 7.0.5-10 (2017) |
| Font Support | ✅ Full DejaVu family | ❌ No fonts |
| Font Configuration | ✅ Complete type.xml | ❌ Missing |
| Security Policy | ✅ Modern, restrictive | ⚠️ Basic |
| Error Handling | ✅ Robust | ⚠️ Basic |
| Caching | ✅ Smart caching | ✅ Basic caching |
| Active Maintenance | ✅ Active | ❌ Archived (2021) |

## Contributing

Issues and pull requests are welcome! Please ensure any changes:

1. Maintain compatibility with the latest Heroku stack
2. Include proper error handling
3. Update documentation as needed
4. Test with actual font rendering operations

## License

MIT License - see LICENSE file for details.

## Support

If you encounter issues:

1. Check the troubleshooting section above
2. Review Heroku build logs for specific error messages  
3. Open an issue with full error details and your use case
4. Include your ImageMagick commands and expected vs actual behavior

---

**Note**: This buildpack is designed specifically for Heroku. For other platforms, consider using system package managers or Docker with appropriate base images.