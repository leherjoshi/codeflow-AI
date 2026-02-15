# Requirements Document

## Introduction

CodeFlow AI is India's first AI-powered competitive programming mentor platform designed to transform chaotic LeetCode practice into structured mastery. The platform addresses the critical gap in Indian coding education where 90% of 1.5M+ engineering graduates fail to crack top tech companies due to unstructured practice. By leveraging AWS AI services (Bedrock Claude 3 Sonnet, Amazon Q), the system analyzes LeetCode profiles, identifies weak topics, and provides personalized learning paths with intelligent hints that guide without spoiling solutions.

## Glossary

- **CodeFlow System**: The complete web application including frontend, backend, AI services, and database
- **LeetCode Profile**: User's public competitive programming statistics and submission history on LeetCode platform
- **Learning Path**: AI-generated personalized sequence of problems targeting user's weak topics
- **Weak Topic**: Programming concept (e.g., Dynamic Programming, Graphs) where user has <40% success rate
- **Strong Topic**: Programming concept where user has >70% success rate
- **Goldilocks Problem**: Next recommended problem with difficulty matching user's current skill level (not too easy/hard)
- **Logic Explainer**: Amazon Q-powered hint system that provides conceptual guidance without code spoilers
- **Skill Heatmap**: Visual dashboard showing proficiency levels across all programming topics
- **Streak Badge**: Gamification element tracking consecutive days of problem-solving activity
- **Student User**: Primary user persona - engineering student from Tier-2/3 colleges
- **Admin User**: Platform administrator managing system health and user analytics
- **AWS Bedrock**: Amazon's managed AI service providing Claude 3 Sonnet model access
- **Amazon Q Developer**: AWS AI assistant for code explanation and logic hints
- **DynamoDB**: AWS NoSQL database service for serverless data storage
- **FastAPI**: Modern Python web framework for building REST APIs
- **Scraping Service**: Backend component extracting real-time data from LeetCode public profiles

## Requirements

### Requirement 1: User Onboarding

**User Story:** As a student user, I want to register with my LeetCode username, so that the system can analyze my coding profile and create a personalized learning path.

#### Acceptance Criteria

1. WHEN a student user submits a valid LeetCode username THEN the CodeFlow System SHALL create a new user account with that username
2. WHEN the CodeFlow System receives a LeetCode username THEN the Scraping Service SHALL fetch the user's problem-solving statistics within 5 seconds
3. IF a LeetCode username does not exist or is invalid THEN the CodeFlow System SHALL display an error message and prevent account creation
4. WHEN user registration completes THEN the CodeFlow System SHALL store user profile data in DynamoDB with timestamp metadata
5. WHERE a user prefers Hindi language THEN the CodeFlow System SHALL set the interface language to Hindi for all subsequent interactions

### Requirement 2: LeetCode Profile Analysis

**User Story:** As a student user, I want the system to analyze my LeetCode submission history, so that I can understand my strengths and weaknesses across different topics.

#### Acceptance Criteria

1. WHEN the Scraping Service fetches LeetCode data THEN the CodeFlow System SHALL parse submission history and categorize problems by topic tags
2. WHEN calculating topic proficiency THEN the CodeFlow System SHALL compute success rate as (solved problems / attempted problems) per topic
3. WHEN a topic has success rate below 40% THEN the CodeFlow System SHALL classify it as a Weak Topic
4. WHEN a topic has success rate above 70% THEN the CodeFlow System SHALL classify it as a Strong Topic
5. WHEN analysis completes THEN the CodeFlow System SHALL generate a Skill Heatmap with color-coded proficiency levels for all topics

### Requirement 3: AI-Powered Learning Path Generation

**User Story:** As a student user, I want an AI-generated personalized learning path, so that I can focus on improving my weak areas systematically.

#### Acceptance Criteria

1. WHEN user profile analysis completes THEN the CodeFlow System SHALL invoke AWS Bedrock Claude 3 Sonnet to generate a Learning Path
2. WHEN generating a Learning Path THEN AWS Bedrock SHALL prioritize Weak Topics and create a problem sequence of 20-30 problems
3. WHEN AWS Bedrock generates a Learning Path THEN the CodeFlow System SHALL receive the response within 10 seconds
4. WHEN a Learning Path is created THEN the CodeFlow System SHALL store it in DynamoDB with user association and creation timestamp
5. WHEN displaying a Learning Path THEN the CodeFlow System SHALL show problem titles, difficulty levels, and estimated completion time

### Requirement 4: Smart Problem Recommendation Engine

**User Story:** As a student user, I want the system to recommend the next problem that matches my current skill level, so that I stay challenged without being overwhelmed.

#### Acceptance Criteria

1. WHEN a user requests the next problem THEN the CodeFlow System SHALL select a Goldilocks Problem from the Learning Path
2. WHEN selecting a Goldilocks Problem THEN the CodeFlow System SHALL consider user's recent success rate and adjust difficulty accordingly
3. WHEN a user completes a problem successfully THEN the CodeFlow System SHALL update topic proficiency scores in real-time
4. WHEN a user fails a problem twice THEN the CodeFlow System SHALL recommend an easier problem on the same topic
5. WHEN all problems in a Weak Topic are completed THEN the CodeFlow System SHALL regenerate the Learning Path with updated priorities

### Requirement 5: AI Logic Explainer with Hints

**User Story:** As a student user, I want to receive conceptual hints when stuck on a problem, so that I can learn the approach without getting the solution spoiled.

#### Acceptance Criteria

1. WHEN a user requests a hint for a problem THEN the CodeFlow System SHALL invoke Amazon Q Developer with problem context
2. WHEN Amazon Q Developer generates a hint THEN the Logic Explainer SHALL provide conceptual guidance without revealing code implementation
3. WHEN a hint is requested THEN the CodeFlow System SHALL deliver the response within 3 seconds
4. WHEN a user requests multiple hints for the same problem THEN the Logic Explainer SHALL provide progressively detailed hints (maximum 3 levels)
5. IF Amazon Q Developer service is unavailable THEN the CodeFlow System SHALL display a fallback hint from pre-generated hint database

### Requirement 6: Visual Skill Dashboard

**User Story:** As a student user, I want to view my progress through an interactive dashboard, so that I can track my improvement over time.

#### Acceptance Criteria

1. WHEN a user accesses the dashboard THEN the CodeFlow System SHALL display the Skill Heatmap with all programming topics
2. WHEN displaying the Skill Heatmap THEN the CodeFlow System SHALL use color gradients (red for weak, yellow for moderate, green for strong)
3. WHEN a user views topic details THEN the CodeFlow System SHALL show problem count, success rate, and recent attempts
4. WHEN dashboard loads THEN the CodeFlow System SHALL fetch and display data within 2 seconds
5. WHEN a user completes a new problem THEN the CodeFlow System SHALL update the dashboard in real-time without page refresh

### Requirement 7: Progress Tracking and Gamification

**User Story:** As a student user, I want to earn streak badges and track my consistency, so that I stay motivated to practice daily.

#### Acceptance Criteria

1. WHEN a user solves at least one problem in a day THEN the CodeFlow System SHALL increment the user's daily streak counter
2. WHEN a user misses a day of practice THEN the CodeFlow System SHALL reset the streak counter to zero
3. WHEN a user achieves streak milestones (7, 30, 100 days) THEN the CodeFlow System SHALL award Streak Badges
4. WHEN displaying progress THEN the CodeFlow System SHALL show total problems solved, current streak, and badges earned
5. WHEN a user views their profile THEN the CodeFlow System SHALL display a calendar heatmap of daily activity

### Requirement 8: Multi-Language Support

**User Story:** As a student user from a Tier-2/3 college, I want to use the platform in Hindi, so that I can understand concepts in my preferred language.

#### Acceptance Criteria

1. WHEN a user selects Hindi language THEN the CodeFlow System SHALL translate all UI elements to Hindi
2. WHEN generating hints in Hindi mode THEN AWS Bedrock SHALL provide explanations in Hindi language
3. WHEN a user switches language THEN the CodeFlow System SHALL persist the preference in DynamoDB
4. WHEN displaying problem descriptions THEN the CodeFlow System SHALL show original English text with optional Hindi translation
5. THE CodeFlow System SHALL support English and Hindi languages at launch

### Requirement 9: Real-Time LeetCode Synchronization

**User Story:** As a student user, I want my LeetCode progress to sync automatically, so that my dashboard always reflects my latest achievements.

#### Acceptance Criteria

1. WHEN a user logs in THEN the Scraping Service SHALL fetch latest LeetCode statistics
2. WHEN the Scraping Service detects new submissions THEN the CodeFlow System SHALL update user profile and recalculate topic proficiency
3. WHEN LeetCode rate limits are encountered THEN the Scraping Service SHALL implement exponential backoff retry logic
4. WHEN scraping fails after 3 retries THEN the CodeFlow System SHALL use cached data and notify the user
5. THE Scraping Service SHALL respect LeetCode's robots.txt and implement rate limiting of maximum 10 requests per minute per user

### Requirement 10: Admin Analytics Dashboard

**User Story:** As an admin user, I want to monitor platform health and user engagement metrics, so that I can ensure system reliability and identify growth opportunities.

#### Acceptance Criteria

1. WHEN an admin user accesses the analytics dashboard THEN the CodeFlow System SHALL display daily active users (DAU), weekly active users (WAU), and monthly active users (MAU)
2. WHEN displaying system health THEN the CodeFlow System SHALL show API response times, error rates, and AWS service status
3. WHEN an admin views user metrics THEN the CodeFlow System SHALL show retention rates, average problems solved per user, and streak distribution
4. WHEN system errors exceed threshold (>5% error rate) THEN the CodeFlow System SHALL send alerts to admin users
5. THE CodeFlow System SHALL refresh analytics data every 5 minutes

### Requirement 11: Performance and Scalability

**User Story:** As a student user, I want the platform to load quickly and handle high traffic, so that I can access it reliably during peak hours.

#### Acceptance Criteria

1. WHEN a user loads any page THEN the CodeFlow System SHALL deliver initial content within 2 seconds
2. WHEN the CodeFlow System serves API requests THEN the response time SHALL be under 500 milliseconds for 95% of requests
3. WHEN concurrent users reach 1000 THEN the CodeFlow System SHALL maintain performance without degradation
4. WHEN AWS Lambda functions execute THEN cold start time SHALL not exceed 1 second
5. THE CodeFlow System SHALL achieve 99.9% uptime measured monthly

### Requirement 12: Error Handling and Resilience

**User Story:** As a student user, I want the system to handle errors gracefully, so that temporary issues don't disrupt my learning experience.

#### Acceptance Criteria

1. WHEN AWS Bedrock service is unavailable THEN the CodeFlow System SHALL display a user-friendly error message and suggest retry
2. WHEN DynamoDB operations fail THEN the CodeFlow System SHALL retry with exponential backoff up to 3 attempts
3. WHEN the Scraping Service encounters invalid HTML THEN the CodeFlow System SHALL log the error and use cached profile data
4. WHEN API rate limits are exceeded THEN the CodeFlow System SHALL queue requests and process them when limits reset
5. WHEN critical errors occur THEN the CodeFlow System SHALL log to CloudWatch and trigger Sentry alerts

### Requirement 13: Data Privacy and Security

**User Story:** As a student user, I want my data to be secure and private, so that I can trust the platform with my learning information.

#### Acceptance Criteria

1. WHEN storing user data THEN the CodeFlow System SHALL encrypt sensitive information at rest in DynamoDB
2. WHEN transmitting data THEN the CodeFlow System SHALL use HTTPS/TLS encryption for all API communications
3. WHEN a user requests account deletion THEN the CodeFlow System SHALL remove all personal data within 24 hours
4. THE CodeFlow System SHALL not share user data with third parties without explicit consent
5. WHEN accessing admin features THEN the CodeFlow System SHALL require authentication with role-based access control

### Requirement 14: Mobile Responsiveness

**User Story:** As a student user, I want to access the platform on my mobile device, so that I can practice coding on the go.

#### Acceptance Criteria

1. WHEN a user accesses the platform on mobile THEN the CodeFlow System SHALL display a responsive layout optimized for screen sizes 320px and above
2. WHEN viewing the Skill Heatmap on mobile THEN the CodeFlow System SHALL adapt the visualization for touch interactions
3. WHEN a user navigates on mobile THEN the CodeFlow System SHALL provide touch-friendly buttons with minimum 44px tap targets
4. WHEN loading on mobile networks THEN the CodeFlow System SHALL optimize asset delivery to reduce data usage
5. THE CodeFlow System SHALL support iOS Safari, Android Chrome, and mobile Firefox browsers

### Requirement 15: Free Access for Students

**User Story:** As a student user from a Tier-2/3 college, I want to use all platform features for free, so that financial constraints don't limit my learning.

#### Acceptance Criteria

1. THE CodeFlow System SHALL provide all core features at no cost to student users
2. WHEN a user registers THEN the CodeFlow System SHALL not require payment information
3. WHEN AWS costs are incurred THEN the CodeFlow System SHALL operate within free tier limits or utilize AWS Bharat cloud credits
4. THE CodeFlow System SHALL display no advertisements that disrupt the learning experience
5. WHEN the platform scales beyond 10,000 users THEN the CodeFlow System SHALL maintain free access through sponsorships or grants

### Requirement 16: Edge Case Handling for LeetCode Integration

**User Story:** As a student user, I want the system to handle unusual LeetCode account scenarios gracefully, so that I can still use the platform even with edge cases.

#### Acceptance Criteria

1. WHEN a LeetCode username returns 404 error THEN the CodeFlow System SHALL display a message indicating the username does not exist and suggest verification
2. WHEN a LeetCode account is banned or suspended THEN the CodeFlow System SHALL detect the status and notify the user that profile analysis is unavailable
3. WHEN a LeetCode account has zero solved problems THEN the CodeFlow System SHALL create a beginner-focused Learning Path with easy problems
4. WHEN the Scraping Service encounters corrupted or malformed LeetCode data THEN the CodeFlow System SHALL log the error and request manual verification
5. WHEN LeetCode returns IP block errors THEN the Scraping Service SHALL rotate request sources and implement extended backoff delays

### Requirement 17: Advanced Rate Limiting and Quota Management

**User Story:** As a student user, I want the system to handle API rate limits intelligently, so that temporary service restrictions don't prevent me from using the platform.

#### Acceptance Criteria

1. WHEN LeetCode returns 429 rate limit errors THEN the Scraping Service SHALL implement exponential backoff starting at 5 seconds and doubling up to 60 seconds
2. WHEN AWS Bedrock API quota is exceeded THEN the CodeFlow System SHALL queue learning path requests and process them when quota resets
3. WHEN AWS Bedrock returns 503 service unavailable THEN the CodeFlow System SHALL retry up to 3 times with 10-second delays
4. WHEN DynamoDB provisioned throughput is exceeded THEN the CodeFlow System SHALL implement adaptive retry with jitter to prevent thundering herd
5. WHEN multiple rate limits are active simultaneously THEN the CodeFlow System SHALL prioritize critical operations (user login, dashboard) over background tasks (analytics)

### Requirement 18: Network Resilience and Timeout Handling

**User Story:** As a student user, I want the system to handle network issues gracefully, so that temporary connectivity problems don't disrupt my learning session.

#### Acceptance Criteria

1. WHEN network timeout occurs during LeetCode scraping THEN the Scraping Service SHALL retry with increased timeout (5s, 10s, 15s)
2. WHEN network timeout occurs during AWS Bedrock invocation THEN the CodeFlow System SHALL cancel the request after 10 seconds and display retry option
3. WHEN network timeout occurs during DynamoDB operations THEN the CodeFlow System SHALL retry with exponential backoff up to 3 attempts
4. WHEN persistent network failures are detected THEN the CodeFlow System SHALL enter offline mode and use cached data exclusively
5. WHEN network connectivity is restored THEN the CodeFlow System SHALL automatically sync pending updates and refresh cached data

### Requirement 19: Language Preference with Fallback

**User Story:** As a student user, I want the system to handle language preferences intelligently, so that I always see content even if translations are unavailable.

#### Acceptance Criteria

1. WHEN a user selects Hindi language preference THEN the CodeFlow System SHALL attempt to display all content in Hindi
2. WHEN Hindi translation is unavailable for specific content THEN the CodeFlow System SHALL display English content with a language indicator
3. WHEN AWS Bedrock fails to generate Hindi hints THEN the CodeFlow System SHALL provide English hints with automatic translation attempt
4. WHEN a user switches from Hindi to English THEN the CodeFlow System SHALL immediately update all UI elements without page refresh
5. WHEN displaying mixed-language content THEN the CodeFlow System SHALL clearly indicate which language each section is in

### Requirement 20: Beginner User Support

**User Story:** As a student user with minimal LeetCode experience, I want the system to provide appropriate guidance, so that I can start my coding journey effectively.

#### Acceptance Criteria

1. WHEN a student user has zero solved problems THEN the CodeFlow System SHALL generate a beginner Learning Path with 100% easy problems
2. WHEN a beginner user requests hints THEN the Logic Explainer SHALL provide more detailed explanations with basic concept reviews
3. WHEN a beginner user fails a problem three times THEN the CodeFlow System SHALL offer tutorial resources and simpler alternative problems
4. WHEN displaying the Skill Heatmap for beginners THEN the CodeFlow System SHALL show encouraging messages and learning resources for each topic
5. WHEN a beginner completes their first problem THEN the CodeFlow System SHALL award a special "First Steps" badge and celebration animation

### Requirement 21: Admin User Analytics Dashboard

**User Story:** As an admin user, I want to view detailed user analytics, so that I can understand user behavior and optimize the platform.

#### Acceptance Criteria

1. WHEN an admin accesses the user analytics dashboard THEN the CodeFlow System SHALL display user growth charts (daily signups, cumulative users)
2. WHEN viewing user analytics THEN the CodeFlow System SHALL show user segmentation by college tier, problem-solving activity, and engagement level
3. WHEN an admin filters analytics by date range THEN the CodeFlow System SHALL update all metrics to reflect the selected period
4. WHEN displaying user cohorts THEN the CodeFlow System SHALL show retention curves for weekly cohorts over 12 weeks
5. WHEN an admin exports analytics data THEN the CodeFlow System SHALL generate CSV files with user metrics and activity logs

### Requirement 22: Admin Content Moderation

**User Story:** As an admin user, I want to moderate user-generated content and handle abuse reports, so that I can maintain platform quality and safety.

#### Acceptance Criteria

1. WHEN an admin accesses the moderation queue THEN the CodeFlow System SHALL display flagged user profiles and reported content
2. WHEN an admin reviews a flagged profile THEN the CodeFlow System SHALL show user activity history, problem attempts, and community interactions
3. WHEN an admin suspends a user account THEN the CodeFlow System SHALL prevent login and display suspension reason to the user
4. WHEN an admin approves flagged content THEN the CodeFlow System SHALL remove it from the moderation queue and log the decision
5. WHEN suspicious activity is detected THEN the CodeFlow System SHALL automatically flag accounts for admin review

### Requirement 23: Admin AWS Cost Monitoring

**User Story:** As an admin user, I want to monitor AWS service costs in real-time, so that I can ensure the platform operates within budget.

#### Acceptance Criteria

1. WHEN an admin accesses the cost monitoring dashboard THEN the CodeFlow System SHALL display current month AWS costs by service (Lambda, DynamoDB, Bedrock)
2. WHEN AWS costs exceed 80% of monthly budget THEN the CodeFlow System SHALL send alert notifications to admin users
3. WHEN viewing cost trends THEN the CodeFlow System SHALL show daily cost breakdown and projected month-end total
4. WHEN an admin analyzes cost drivers THEN the CodeFlow System SHALL display top cost-generating users and API endpoints
5. WHEN cost optimization opportunities are identified THEN the CodeFlow System SHALL suggest actions (caching, rate limiting, resource scaling)

### Requirement 24: Admin A/B Testing for Recommendations

**User Story:** As an admin user, I want to run A/B tests on problem recommendation algorithms, so that I can optimize learning outcomes.

#### Acceptance Criteria

1. WHEN an admin creates an A/B test THEN the CodeFlow System SHALL allow configuration of test variants (algorithm parameters, problem selection criteria)
2. WHEN an A/B test is active THEN the CodeFlow System SHALL randomly assign users to control or treatment groups
3. WHEN displaying A/B test results THEN the CodeFlow System SHALL show key metrics (completion rate, time to solve, user satisfaction) for each variant
4. WHEN an A/B test concludes THEN the CodeFlow System SHALL provide statistical significance analysis and recommend winning variant
5. WHEN a winning variant is selected THEN the CodeFlow System SHALL allow one-click rollout to all users

### Requirement 25: Admin Bulk User Import

**User Story:** As an admin user, I want to import multiple users from college WhatsApp groups, so that I can onboard entire student cohorts efficiently.

#### Acceptance Criteria

1. WHEN an admin uploads a CSV file with LeetCode usernames THEN the CodeFlow System SHALL validate and import all users in batch
2. WHEN processing bulk import THEN the CodeFlow System SHALL handle rate limits by queuing scraping requests and processing them sequentially
3. WHEN bulk import completes THEN the CodeFlow System SHALL generate a report showing successful imports, failures, and invalid usernames
4. WHEN importing users from a college group THEN the CodeFlow System SHALL tag users with college affiliation for cohort analytics
5. WHEN bulk import encounters errors THEN the CodeFlow System SHALL continue processing remaining users and log failures for manual review
