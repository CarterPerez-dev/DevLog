# achievements

**Type:** File Overview
**Repository:** CertGames-Core
**File:** backend/devtools/data/achievements/achievements.js
**Language:** javascript
**Lines:** 1-519
**Complexity:** 0.0

---

## Source Code

```javascript
db.achievements.insertMany([
  {
    achievementId: 'test_rookie',
    title: 'Test Rookie',
    description: 'Complete your first test. Welcome to the grind!',
    category: 'test',
    criteria: { testCount: 1 },
    xpReward: 50,
    coinReward: 25
  },
  {
    achievementId: 'bronze_grinder',
    title: 'Bronze Grinder',
    description: 'Complete 5 tests. You are putting in the work!',
    category: 'test',
    criteria: { testCount: 5 },
    xpReward: 100,
    coinReward: 50
  },
  {
    achievementId: 'test_enthusiast',
    title: 'Test Enthusiast',
    description: 'Complete 10 tests. Building momentum!',
    category: 'test',
    criteria: { testCount: 10 },
    xpReward: 150,
    coinReward: 75
  },
  {
    achievementId: 'silver_scholar',
    title: 'Silver Scholar',
    description: 'Complete 25 tests. Starting to look like a pro!',
    category: 'test',
    criteria: { testCount: 25 },
    xpReward: 250,
    coinReward: 125
  },
  {
    achievementId: 'gold_god',
    title: 'Gold God',
    description: 'Complete 50 tests. Unstoppable!',
    category: 'test',
    criteria: { testCount: 50 },
    xpReward: 500,
    coinReward: 250
  },
  {
    achievementId: 'platinum_pro',
    title: 'Platinum Pro',
    description: 'Complete 100 tests. No life, just tests!',
    category: 'test',
    criteria: { testCount: 100 },
    xpReward: 750,
    coinReward: 500
  },
  {
    achievementId: 'test_legend',
    title: 'Test Legend',
    description: 'Complete 200 tests. You have transcended reality!',
    category: 'test',
    criteria: { testCount: 200 },
    xpReward: 1000,
    coinReward: 750
  },

  // ============= MILESTONE CATEGORY =============
  {
    achievementId: 'level_up_5',
    title: 'Rising Star (Level 5)',
    description: 'Reach Level 5. Just getting started!',
    category: 'milestone',
    criteria: { level: 5 },
    xpReward: 100,
    coinReward: 100
  },
  {
    achievementId: 'level_up_10',
    title: 'Double Digits (Level 10)',
    description: 'Reach Level 10. Now we are talking!',
    category: 'milestone',
    criteria: { level: 10 },
    xpReward: 200,
    coinReward: 150
  },
  {
    achievementId: 'level_up_15',
    title: 'Skilled Learner (Level 15)',
    description: 'Reach Level 15. Your dedication shows!',
    category: 'milestone',
    criteria: { level: 15 },
    xpReward: 300,
    coinReward: 200
  },
  {
    achievementId: 'level_up_20',
    title: 'Expert in Training (Level 20)',
    description: 'Reach Level 20. Serious commitment!',
    category: 'milestone',
    criteria: { level: 20 },
    xpReward: 400,
    coinReward: 250
  },
  {
    achievementId: 'mid_tier_grinder_25',
    title: 'Mid-Tier Grinder (Level 25)',
    description: 'Reach Level 25. You are in deep now!',
    category: 'milestone',
    criteria: { level: 25 },
    xpReward: 500,
    coinReward: 300
  },
  {
    achievementId: 'level_up_30',
    title: 'Cybersecurity Specialist (Level 30)',
    description: 'Reach Level 30. Professional le
```

---

## File Overview

# achievements.js Documentation

## Purpose and Responsibility
This JavaScript file is responsible for defining a collection of achievements within the CertGames-Core backend, specifically for the development tools. Each achievement includes details such as its ID, title, description, category, criteria for completion, and rewards in experience points (XP) and coins.

## Key Exports or Public Interface
The file does not explicitly export any functions or objects; instead, it inserts a large array of achievement documents directly into the database using `insertMany`. This approach ensures that all achievements are pre-defined and ready to be used within the application without needing additional setup or configuration.

## How It Fits in the Project
This file is crucial for initializing the game's achievements system. By populating the database with predefined achievements, it sets up a structured way to track player progress and reward them appropriately based on their actions within the game. This data is essential for the backend logic that handles user achievements, leaderboards, and rewards.

## Notable Design Decisions
- **Predefined Data**: The use of `insertMany` directly in the file ensures that all necessary achievements are available from the start, reducing the need for dynamic generation or external configuration.
- **Category-Based Structure**: Achievements are categorized into 'test', 'milestone', and 'collection' to organize them logically. This structure helps in managing different types of achievements more effectively.
- **Criteria and Rewards**: Each achievement has specific criteria (e.g., testCount, level, coins) that determine when it can be claimed, along with corresponding XP and coin rewards, providing a clear incentive system for players.

---

*Generated by CodeWorm on 2026-03-12 11:14*
