---
name: trip-planner
description: AI travel planner that searches flights, hotels, and attractions across multiple platforms (Google Flights, Kayak, Booking.com, Airbnb, TripAdvisor), generates day-by-day itineraries, syncs to Google Calendar, and tracks budgets in Google Sheets.
---

# AI Trip Planner

Plan complete trips by searching across multiple travel platforms simultaneously. Compare flights, find hotels, discover attractions, build day-by-day itineraries, sync to calendar, and track budgets.

## When to Use
- "Plan a 5-day trip to Tokyo for 2 people under $3000"
- "Find the cheapest flights from SF to London in July"
- "Compare hotels near Times Square for next weekend"
- "Build a complete itinerary for a family trip to Italy"

## Required Context (ask if not provided)
- **Destination(s)**: Where are they going?
- **Dates**: Travel dates (or flexible date range)
- **Travelers**: Number of people, ages, any special needs
- **Budget**: Total budget or per-day budget
- **Priorities**: Adventure? Relaxation? Culture? Food? Family-friendly?
- **Accommodation preference**: Hotel, Airbnb, hostel, luxury?

## Platform Integrations
| Platform | Usage |
|---|---|
| Google Flights (Browser) | Flight search and comparison |
| Kayak (Browser) | Flight + hotel meta-search |
| Skyscanner (Browser) | Budget flight discovery |
| Booking.com (Browser) | Hotel search with reviews |
| Airbnb (Browser) | Short-term rental search |
| TripAdvisor (Browser) | Attraction ratings and reviews |
| Google Maps (Browser) | Distance/route optimization |
| Google Calendar | Itinerary sync (daily events) |
| Google Sheets | Budget tracker |
| Google Docs | Exportable trip plan document |

## Safety Rules
- **Read-only on all travel platforms** - never book anything without explicit user approval
- **Wait 5-10 seconds between page loads** on travel sites
- **Maximum 30 searches per platform per session**
- **No account creation** on any platform
- **Always present prices in user's preferred currency**

## Workflow A: Full Trip Planning

### Phase 1: Flight Search (if applicable)
```javascript
// Search Google Flights
var page = await browser.newtab("https://www.google.com/travel/flights")
await page.wait({ waitTime: 3 })
// Enter: origin, destination, dates, passengers
// Capture: airline, times, stops, price for top 5-8 options

// Cross-reference with Kayak
await page.goto({ url: "https://www.kayak.com/flights/[origin]-[dest]/[dates]" })
await page.wait({ waitTime: 5 })
// Capture: additional options, price alerts

// Cross-reference with Skyscanner for budget options
await page.goto({ url: "https://www.skyscanner.com/transport/flights/[origin]/[dest]/[dates]" })
```

**Output**: Comparison table of top 5 flight options across platforms.

### Phase 2: Accommodation Search
```javascript
// Search Booking.com
await page.goto({ url: "https://www.booking.com/searchresults.html?ss=[destination]&checkin=[date]&checkout=[date]&group_adults=[n]" })
// Filter by: price range, rating 8+, location
// Capture: name, price/night, rating, location, amenities

// Search Airbnb
await page.goto({ url: "https://www.airbnb.com/s/[destination]/homes?checkin=[date]&checkout=[date]&adults=[n]" })
// Capture: name, price/night, rating, type, amenities
```

**Output**: Top 5 hotels + Top 5 Airbnbs with price/rating comparison.

### Phase 3: Attraction Discovery
```javascript
// Search TripAdvisor
await page.goto({ url: "https://www.tripadvisor.com/Attractions-[destination]" })
// Sort by: rating, then filter by category (user preference)
// Capture: name, rating, review count, price, hours, location

// Use Google Maps for proximity grouping
// Group attractions by neighborhood for efficient daily routing
```

### Phase 4: Itinerary Generation
Build a day-by-day plan optimized for:
- **Geographic proximity** (minimize daily travel distance)
- **Opening hours** (visit attractions when open)
- **Meal timing** (breakfast, lunch, dinner spots near activities)
- **Energy management** (mix active/relaxing activities)
- **Budget pacing** (spread expensive activities across days)

```
## Day 1: [Neighborhood/Theme]
- 9:00 AM: [Attraction 1] (rating, est. time, cost)
- 11:30 AM: [Lunch spot] (cuisine, price range)
- 1:00 PM: [Attraction 2]
- 4:00 PM: [Coffee/rest break]
- 6:00 PM: [Dinner reservation]
- Transport: [How to get between locations]
- Day budget: $XX
```

### Phase 5: Calendar Sync
```javascript
for (var day of itinerary.days) {
  for (var activity of day.activities) {
    await google.calendar.createEvent({
      summary: activity.name,
      description: activity.details + "\nAddress: " + activity.address + "\nCost: " + activity.cost,
      location: activity.address,
      start: activity.startTime,
      end: activity.endTime
    })
  }
}
```

### Phase 6: Budget Tracker
```javascript
var sheet = await google.sheets.createSpreadsheet({
  title: "Trip Budget - [Destination] - [Dates]",
  sheets: ["Overview", "Flights", "Hotels", "Activities", "Food", "Transport"]
})
// Overview sheet: category totals, budget vs actual
// Each category sheet: individual items with prices
```

### Phase 7: Trip Document Export
```javascript
var doc = await google.docs.createDocument({ title: "Trip Plan: [Destination]" })
// Insert: flight details, hotel confirmation, daily itinerary, restaurant list, emergency contacts, packing list
```

## Workflow B: Quick Flight/Hotel Search Only
Skip the full planning, just compare prices across platforms for a specific search.

## Workflow C: Day Trip / Activity Planning
For users already at a destination who need help planning a single day.

## Output Quality Checks
- All prices verified across at least 2 platforms
- Attraction hours confirmed current
- Travel times between locations estimated via Google Maps
- Total budget calculated and compared to user's limit
- Emergency info included (local embassy, hospital, emergency number)
