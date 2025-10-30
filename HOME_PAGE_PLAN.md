# Home Page Design Plan - Daily Gospel Readings

## Context
Creating a home page for the Daily Gospel Readings website (St. John Lutheran Church). This will be embedded in Google Sites (no html/head/body tags, inline CSS/JS only).

## Key Design Decisions

### 1. Focus & Audience
- **Primary Focus**: Daily Bible reading site for church members and general public
- **Secondary**: Welcoming visitors to St. John Lutheran Church
- **Removed**: Gospel Project-specific content (can be added later as separate page if needed)
- **Goal**: Get users to today's reading quickly while providing church connection

### 2. Church Information
**St. John Lutheran Church - Boerne, TX**
- **Tagline**: "Know Jesus…Make Jesus Known"
- **Website**: https://www.stjohnlutheran.com/
- **Worship Times**:
  - 8:00 AM - Traditional Worship
  - 9:30 AM - Praise Worship
  - 11:00 AM - Traditional Worship
- **Livestream**: Available online
- **Location**: 315 Rosewood Avenue, Boerne, TX 78006
- **Phone**: (830) 249-3651
- **Email**: info@stjohnlutheran.com

### 3. Design Style
- Match the clean, modern aesthetic of daily reading page template
- Color palette: Teal accent (#72abbf), neutral grays, warm tones
- Serif headings (matching ESV Bible style)
- Plenty of white space
- Responsive mobile design
- Subtle background image in hero section (light texture or faded Bible/cross)

## Page Structure

### Section 1: Hero Section
**Content:**
```
Daily Gospel Readings
St. John Lutheran Church

[Tagline] Know Jesus…Make Jesus Known

Join us each day as we journey through Scripture together. Whether you're at home,
on your lunch break, or winding down in the evening, find today's reading beautifully
presented with images, audio, and reflections to deepen your walk with Christ.

[Large Button: Today's Reading]  [Button: Browse All Readings]
```

**Design:**
- Centered layout
- Subtle background (light texture or faded image with overlay)
- Church logo above church name (if logo available)
- Prominent CTA buttons

### Section 2: What You'll Find Each Day
**Layout:** 4-column icon grid (2×2 on mobile)

**Content:**
```
[📖] Scripture Reading
Read the full ESV Bible text with optional footnotes and cross-references.

[🎵] Audio Recording
Listen to the passage read aloud—perfect for your commute or morning routine.

[🖼️] Visual Inspiration
Each reading features a verse card or themed image to help you reflect and meditate.

[✝️] Daily Devotional
Apply God's Word to your life with thoughtful reflections and commentary.
```

### Section 3: Join Us for Worship
**Layout:** Two-column (stacks on mobile)

**Left Column - Worship Times:**
```
Worship with Us

Sunday Services:
• 8:00 AM - Traditional Worship
• 9:30 AM - Praise Worship
• 11:00 AM - Traditional Worship

Can't make it in person? Join us online for our livestreamed services.

[Button: Watch Online]  [Link: Plan Your Visit]
```

**Right Column - Location:**
```
Find Us

315 Rosewood Avenue
Boerne, TX 78006

Phone: (830) 249-3651
Email: info@stjohnlutheran.com

[Link to directions or small map]
```

### Section 4: Call to Action Footer
**Content:**
```
Ready to Begin Your Journey?

Start today and make God's Word a daily priority in your life.

[Button: Go to Today's Reading]
```

## Logo Considerations

**Options:**
1. Use St. John Lutheran Church logo in hero section (if available)
2. Create simple text-based logo with icon (📖 Daily Gospel Readings)
3. Wait for user to provide logo file

**Recommendation**: Place church logo above "St. John Lutheran Church" text in hero (100-150px width)

## Technical Notes

- **File Location**: `c:\pyApps\sjlc-private\testing\test-home-page.html`
- **Google Sites Requirements**:
  - No `<html>`, `<head>`, or `<body>` tags
  - All CSS in `<style>` block
  - All JavaScript in `<script>` block
  - Embedded element only
- **Reference Files**:
  - Daily reading template: `c:\pyApps\sjlc-private\testing\test-daily-reading.html`
  - CSS variables and styling patterns to maintain consistency

## Open Questions (for user)

1. Banner image colors/imagery for complementary design?
2. Church logo file available or design without?
3. Livestream URL for "Watch Online" button?
4. Embedded map or just "Get Directions" link?
5. Navigation button destinations (placeholder # or specific URLs)?

## Color Palette (from daily reading page)
```css
--bible-serif-font: "Sentinel", "Gentium Plus", Georgia, Times, serif;
--bible-sans-font: "Gotham", Helvetica, Arial, sans-serif;
--bible-text-color: #424547;
--bible-secondary-color: #5e6266;
--bible-tertiary-color: #9a9fa3;
--bible-link-color: #72abbf;
--accent-color: #72abbf;
```

## Next Steps
1. Answer open questions with user
2. Create `test-home-page.html` in testing directory
3. Test responsive design
4. Review with user for feedback/adjustments
5. Update claude.md with home page documentation
