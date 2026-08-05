# Colors

Source: https://admin-ui.vercel.app/foundations/colors

Admin UI color palettes follow WCAG AA contrast ratios. Consume colors through design tokens ([design-tokens.md](design-tokens.md)), not raw hex values.

## Color tones (semantic functions)

| Tone | Purpose |
|------|---------|
| Main | Principal interactive elements directing attention to primary tasks |
| Critical | Errors or situations requiring extreme caution |
| Warning | Unexpected behavior, requires user caution |
| Positive | Favorable outcomes or new accomplishments |
| Neutral | Non-emphasized information from completed past actions |
| Info | Calm, non-urgent guidance and information |

Semantic mappings by context:

- Actions: primary
- Form fields: invalid, valid
- Messages: error, warning, success, description, notification
- Status: removed, pending, added, default
- Priority: major, medium, trivial, minor
- Payments: cancelled, in progress, completed, archived

## Palettes (05 = lightest, 60 = darkest)

### Blue — primary actions

| Name | HEX |
|------|-----|
| blue05 | #F1F4FC |
| blue10 | #DDE5F7 |
| blue20 | #BBCBF0 |
| blue30 | #96B2F2 |
| blue40 | #2953B2 |
| blue50 | #20408C |
| blue60 | #172F66 |

Use blue40 for primary actions, buttons, interactive elements. Avoid it for information highlighting or backgrounds; it interferes with visual priorities on the page.

### Green — positive

| Name | HEX |
|------|-----|
| green05 | #ECF7EC |
| green10 | #DFF1E0 |
| green20 | #AEDDB0 |
| green30 | #83D187 |
| green40 | #38853C |
| green50 | #285F2B |
| green60 | #1C431E |

Positive messages, completed states, progress indicators. Avoid for "next" buttons.

### Orange — warning

| Name | HEX |
|------|-----|
| orange05 | #FFF5EC |
| orange10 | #FFEBD7 |
| orange20 | #FFCD9B |
| orange30 | #F5AE70 |
| orange40 | #D56A00 |
| orange50 | #A65200 |
| orange60 | #5C2C00 |

Warning messages/states. Use orange50 for text on white backgrounds.

### Red — critical

| Name | HEX |
|------|-----|
| red05 | #FBF2F2 |
| red10 | #F8E3E3 |
| red20 | #EDBABA |
| red30 | #F49494 |
| red40 | #CC3D3D |
| red50 | #962828 |
| red60 | #782020 |

Critical messages, deletions, failure states, critical action buttons.

### Light Blue — info

| Name | HEX |
|------|-----|
| lightBlue05 | #EDF5FB |
| lightBlue10 | #DEECF7 |
| lightBlue20 | #B9DAF3 |
| lightBlue30 | #7CBBEA |
| lightBlue40 | #2978B5 |
| lightBlue50 | #216090 |
| lightBlue60 | #143C5C |

Information messages and status indicators; informational text/number highlighting; line charts.

### Gray — text, backgrounds, borders

| Name | HEX |
|------|-----|
| white | #FFFFFF |
| gray05 | #F4F4F4 |
| gray10 | #E6E6E6 |
| gray20 | #DDDDDD |
| gray30 | #B9B9B9 |
| gray40 | #747474 |
| gray50 | #545454 |
| gray60 | #3B3B3B |
| black | #000000 |

Black for main text; gray40 for secondary text (captions, subtitles) and icons; white for backgrounds; gray10 for highlighted backgrounds (sidebars); gray20 for borders and dividers.

### Purple

| Name | HEX |
|------|-----|
| purple05 | #F6F3FA |
| purple10 | #EDE8F4 |
| purple20 | #D3C4E4 |
| purple30 | #C6A5EA |
| purple40 | #835AB3 |
| purple50 | #684391 |
| purple60 | #462D63 |

### Cyan

| Name | HEX |
|------|-----|
| cyan05 | #EFF9FB |
| cyan10 | #D5EFF4 |
| cyan20 | #A4DEE9 |
| cyan30 | #63CBDB |
| cyan40 | #1C99A8 |
| cyan50 | #207883 |
| cyan60 | #093F46 |

### Teal

| Name | HEX |
|------|-----|
| teal05 | #E8F7F3 |
| teal10 | #CDEFE7 |
| teal20 | #A6E0D2 |
| teal30 | #66CBB7 |
| teal40 | #007969 |
| teal50 | #006053 |
| teal60 | #00463C |

### Pink — VTEX branding only

| Name | HEX |
|------|-----|
| pink05 | #FBF2F5 |
| pink10 | #F5E0E7 |
| pink20 | #EAB2C4 |
| pink30 | #F48AAB |
| pink40 | #DA3561 |
| pink50 | #B1204B |
| pink60 | #850D35 |

Reserved for brand applications (logos, illustrations); do not use in UI semantics.

## Token structure

Three top-level categories: Background, Foreground (text and icons), Border. Contextual token groups: Actions (Default/Hover/Pressed/Disabled), Overlay (popups, modals, popovers), Structure (sidebar, topbar, page header), Notifications (toast, alert), Fields (input, textarea, select), Controls (checkbox, radio, toggle), Loading (skeleton), List Box Item.
