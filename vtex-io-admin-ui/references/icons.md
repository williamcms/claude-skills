# Icons

Source: https://admin-ui.vercel.app/foundations/icons

Admin UI icons are React components based on Phosphor icon names. In code each icon is named `Icon<Name>` and imported from `@vtex/admin-ui` (backed by the `@vtex/admin-ui-icons` package):

```tsx
import { IconTrash, IconPlus, IconCaretDown, IconArrowsClockwise } from '@vtex/admin-ui'

<IconTrash />
<IconCaretDown csx={{ color: '$secondary' }} />
```

## Props

| Property | Type | Description | Default |
|----------|------|-------------|---------|
| csx | StyleObject | Style properties (token-based) | — |
| weight | `outline` \| `fill` | Icon weight | `outline` |
| size | `small` \| `regular` | Icon size | `regular` |
| mirrored | boolean | Mirrors direction (for RTL/directional icons) | `false` |

All standard `svg` element props are also supported.

## Available icons (base names; prefix with `Icon` in code)

ActivityAlarm, Archive, ArrowCircleUpRight, ArrowCounterClockwise, ArrowDown, ArrowLeft, ArrowLineDown, ArrowLineUp, ArrowRight, ArrowSquareOut, ArrowSquareUpRight, ArrowUUpLeft, ArrowUp, ArrowUpRight, ArrowsClockwise, Barcode, Bell, CalendarBlank, CaretDown, CaretLeft, CaretRight, CaretUp, ChartBar, ChartLineUp, ChatText, Check, CheckCircle, Clock, CloudArrowUp, Code, CopySimple, CornersOut, CreditCard, Cube, CurrencyCircleDollar, DotsSixVertical, DotsThreeVertical, Envelope, Eye, EyeSlash, Faders, FileText, Flag, Folder, GearSix, Gift, GlobeHemisphereWest, HandbagSimple, Headset, Heart, House, IdentificationCard, ImageSquare, Key, Layout, Link, LinkBreak, List, ListDashes, ListNumbers, LockKey, LockKeyOpen, MagnifyingGlass, MagnifyingGlassPlus, MapPin, Megaphone, Minus, Notebook, PaperPlaneTilt, Paperclip, Pencil, Plus, Printer, Question, Receipt, RocketLaunch, Rows, ShareNetwork, ShoppingCartSimple, SignOut, SquaresFour, Stack, Storefront, Tag, TextAlignCenter, TextAlignLeft, TextAlignRight, TextBolder, TextItalic, TextUnderline, Ticket, Trash, Truck, User, Users, Warning, WarningCircle, X, XCircle, XOctagon

## Usage rules

- Icon-only buttons require an `aria-label`.
- Use `mirrored` for directional icons in RTL contexts instead of custom transforms.
- Color icons through `csx` with foreground tokens; icons inherit `currentColor` semantics.
