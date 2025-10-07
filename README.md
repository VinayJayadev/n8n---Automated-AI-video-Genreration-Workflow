# GenAI Video generation Pipeline - Complete Documentation
<img width="2546" height="682" alt="image" src="https://github.com/user-attachments/assets/e4d6a7df-8002-494f-9fb1-ec1723dd1cc9" />

## Overview
This workflow generates AI-powered Christmas-themed video sequences using a multi-stage pipeline. It breaks down storylines into scenes, creates branded video content, and logs results with full reproducibility tracking. The system automates from storyline input to generated video sequences suitable for marketing content creation.

**Purpose**: Automated brand-compliant video generation for VW ID.4 Christmas campaigns  
**Output**: 5-7 video sequences with scene metadata and tracking  
**Integration**: Google Sheets logging, Notion publishing, optional Google Drive storage

---

## Prerequisites
- **KIE.AI Account**: Sign up at [KIE.AI](https://kie.ai/) to obtain your free or paid API key
- **n8n Instance**: Cloud or self-hosted with HTTP Request and form submission capabilities  
- **Google Workspace**: For Google Sheets logging (optional Google Drive upload)
- **Notion Integration**: For storyboard publishing (optional)
- **Basic AI Prompting**: Knowledge of video generation prompts for optimal results

---

## Setup Instructions

### 1. Obtain API Key
- Register at KIE.AI and generate your API key
- Store it securely in n8n credentials—do not share publicly
- Test API access with a simple request to verify authentication

### 2. Configure Credentials
**In n8n Credentials panel:**
- **KIE.AI API**: Add HTTP Basic Auth or API Key credential with your KIE.AI key
- **Google Sheets**: OAuth2 or Service Account for sheet creation/updates  
- **Notion API**: Internal integration token for block publishing
- **Google Drive** (optional): OAuth2 for video file archival

### 3. Configure Workflow Parameters
**Key settings to verify:**
- **Model**: `runway-duration-5-generate` (or preferred Runway model)
- **Aspect Ratio**: `16:9` for landscape video output
- **Duration**: 2-3 seconds per scene (5 seconds total per video)
- **Quality**: `720p` or `1080p` based on requirements
- **Seeds**: Fixed integers for reproducible results

### 4. Test the Workflow
1. Click **"Execute Workflow"** in n8n
2. Fill the chat form with your Christmas storyline
3. Submit and monitor execution through each node
4. Verify outputs in Google Sheets and Notion (if configured)

---

## Workflow Architecture

### Stage 1: Scene Generation
**Nodes**: Chat Message → Prompt Engineer → Edit Fields
- **Input**: User storyline via chat interface
- **Process**: Gemini 2.5 Flash breaks down story into 5-7 structured scenes
- **Output**: Array of scene objects with prompts, durations, and seeds

### Stage 2: Scene Processing  
**Nodes**: Loop Through Scenes → Split Out → Video Generation
- **Process**: Iterate through each scene individually
- **API Call**: POST to KIE.AI Runway endpoint with scene-specific prompts
- **Tracking**: Each scene gets unique ID and generation parameters

### Stage 3: Video Generation & Polling
**Nodes**: Send Request → Wait → Obtain Status → Video Complete?
- **Generation**: Submit video request with branded prompts
- **Polling**: Check status every 10 seconds until completion  
- **Validation**: Verify both video and image URLs are present

### Stage 4: Results Processing & Publishing
**Nodes**: Format Results → Create Sheet → Update Rows → Notion/Drive
- **Logging**: Google Sheets with execution tracking and URLs
- **Publishing**: Notion blocks for storyboard review
- **Archive**: Optional Google Drive upload for asset management

---

## Key Features

### Brand Fidelity Controls
- **VW ID.4 Emphasis**: Every prompt includes "Volkswagen ID.4" branding
- **Consistent Styling**: Christmas theme with snow, lights, suburban settings
- **Negative Prompts**: Excludes non-VW badges and off-model elements
- **Visual Consistency**: Maintains vehicle recognizability across scenes

### Reproducibility System
- **Fixed Seeds**: Each scene uses deterministic seed (scene_id × 12345)
- **Parameter Logging**: Complete tracking of model, prompt, duration, quality
- **Execution IDs**: Unique run identification for result correlation
- **Version Control**: Model versions and API parameters documented

### Error Handling & Resilience
- **Status Polling**: Automatic retry until video generation completes
- **Null Checking**: Validates video/image URLs before proceeding  
- **Timeout Protection**: Reasonable delays to prevent API rate limiting
- **Fallback Logging**: Captures failures for debugging

---

## Node-by-Node Configuration

### Chat Message (Trigger)
```
Purpose: Capture user storyline input
Configuration: Simple text input for Christmas storyline
Output: chat_input field with user narrative
```

### Prompt Engineer (Google Gemini)
```
Model: gemini-2.5-flash
System Prompt: VW ID.4 Christmas video production expert
User Prompt: Scene breakdown with specific JSON structure
Response Format: Structured JSON with scenes array
```

### Edit Fields (Data Processing)
```
Mode: Manual Mapping
Field: clean_scenes = JSON.parse(candidates[0].content.parts[0].text).scenes
Field: chat_input = Original storyline for context
```

### Loop Through Scenes (Batch Processing)
```
Batch Size: 1 (process scenes individually)
Mode: Split into batches for sequential processing
Output: Individual scene objects for video generation
```

### Send Video Generation Request (API Call)
```
Method: POST
URL: https://api.kie.ai/api/v1/runway/generate
Headers: Authorization with KIE.AI API key
Body: JSON with prompt, model, duration, aspect ratio, seeds
```

### Wait for Video Processing (Delay)
```
Duration: 30 seconds (adjust based on typical generation time)
Purpose: Allow initial processing before status check
```

### Obtain Status (Polling)
```
Method: GET  
URL: https://api.kie.ai/api/v1/runway/record-detail
Query: taskId from generation response
Purpose: Check completion status and retrieve URLs
```

### Video Complete? (Validation)
```
Condition: data.videoInfo is not empty
True Branch: Continue to results processing
False Branch: Loop back to status check with retry logic
```

---

## Data Flow & Schema

### Scene Object Structure
```json
{
  "scene_id": 1,
  "visual_description": "Father approaches suburban home on snowy Christmas Eve...",
  "image_prompt": "Wide shot of suburban house, Christmas lights, VW ID.4...",
  "duration": 2.5,
  "seed": 12345
}
```

### API Response Structure
```json
{
  "code": 200,
  "msg": "success", 
  "data": {
    "taskId": "uuid-string",
    "videoInfo": {
      "videoUrl": "https://...",
      "imageUrl": "https://..."
    }
  }
}
```

### Logging Schema
```json
{
  "execution_id": "unique-run-id",
  "scene_id": 1,
  "prompt": "Enhanced scene prompt with branding",
  "video_url": "Generated video URL",
  "image_url": "Thumbnail image URL", 
  "model": "runway-duration-5-generate",
  "seed": 12345,
  "duration": 2.5,
  "timestamp": "2025-10-06T23:26:00Z"
}
```

---

## Customization Options

### Enhanced Prompts
- **Duration Control**: Adjust per-scene timing (2-3 seconds recommended)
- **Style Variations**: Add "cinematic", "commercial", "documentary" styling
- **Camera Movements**: Include "pan", "zoom", "tracking shot" directions  
- **Lighting**: Specify "golden hour", "ambient", "dramatic" lighting
- **Weather Effects**: Control snow intensity, visibility, atmosphere

### Brand Customization  
- **Vehicle Focus**: Ensure ID.4 remains central in every scene
- **Color Palette**: Emphasize VW brand colors and Christmas themes
- **Environment**: Suburban, family-friendly, premium positioning
- **Mood**: Peaceful, magical, sophisticated Christmas atmosphere

### Technical Adjustments
- **Quality Settings**: Balance between 720p/1080p and generation time
- **Aspect Ratios**: 16:9 for web, 9:16 for social media variants
- **Model Selection**: Different Runway models for style variations
- **Batch Sizes**: Parallel processing vs. sequential for system resources

---

## Troubleshooting Guide

### Common Issues

**"JSON parameter needs to be valid JSON"**
- Check expression syntax in HTTP Request body
- Verify field names match exactly (chat_input vs chatInput)
- Use JSON.stringify() for proper string escaping

**"Video Complete?" always returns False**
- Verify videoInfo object structure in API response
- Check for null/empty video URLs in response
- Increase Wait duration if generation takes longer

**Google Sheets errors**
- Confirm sheet permissions and integration access
- Verify dynamic sheet naming expressions
- Check column mapping matches sheet headers

**Notion publishing failures**  
- Ensure page/database is shared with integration
- Verify block structure and property types
- Check file upload vs. external URL handling

### Performance Optimization

**Reduce Generation Time**
- Use lower quality settings for testing
- Implement parallel scene processing if API supports
- Cache successful generations to avoid regeneration

**Cost Management**
- Monitor API usage and implement daily limits
- Use shorter durations for initial testing
- Implement approval gates before final generation

---

## Best Practices

### Prompt Engineering
- Keep VW ID.4 branding consistent across all scenes
- Include specific Christmas elements (snow, lights, decorations)
- Specify camera angles and movements for cinematic quality
- Balance creativity with brand guideline compliance

### Workflow Management
- Test individual nodes before full workflow execution
- Use descriptive execution IDs for run tracking
- Implement proper error logging and notifications
- Schedule regular cleanup of generated assets

### Quality Assurance
- Review generated videos for brand compliance
- Maintain consistent visual style across scenes
- Document successful prompt patterns for reuse
- Collect feedback for continuous improvement

---

## API Rate Limits & Costs

### KIE.AI Credit System
- **Credit Value**: Each credit = $0.005 USD (half a cent)
- **Package Options**: 
  - $5 = 1,000 credits
  - $50 = 10,000 credits (most popular)
  - $500 = 105,000 credits (5% discount)
  - $1,250 = 275,000 credits (10% discount)

### Runway API Costs (via KIE.AI)
- **5-second video (720P)**: 12 credits = $0.06 per video
- **5-second video (1080P)**: 30 credits = $0.15 per video  
- **10-second video (720P)**: 30 credits = $0.15 per video
- **Extend video (5 seconds, 720P)**: 12 credits = $0.06 per extension
- **Extend video (5 seconds, 1080P)**: 30 credits = $0.15 per extension
- **Get video details**: 0 credits (free status checks)

### Cost Estimates for VW Workflow
- **Per scene (5-second 720P)**: $0.06 per video
- **Full 7-scene workflow (720P)**: $0.42 per complete run
- **Full 7-scene workflow (1080P)**: $1.05 per complete run
- **Monthly budget**: 
  - 10 complete runs (720P): ~$4.20
  - 10 complete runs (1080P): ~$10.50
  - 100 complete runs (720P): ~$42.00

### Rate Limits & Generation Time
- **API Rate Limits**: Standard tier supports multiple concurrent requests
- **Generation Time**: 40-60 seconds typical for 5-second (6-7) video
- **Recommended**: Start with $50 package (10,000 credits) for extensive testing and production

### Cost Optimization Tips
- **Use 720P for testing**: 60% cost savings vs 1080P
- **Quality vs. Cost**: Balance video quality needs with budget requirements

---

## Support & Updates

### Documentation Updates
- This workflow is optimized for KIE.AI Runway integration as of October 2025
- API endpoints and parameters may change with provider updates
- Check KIE.AI documentation for latest model availability

### Community Resources
- n8n Community: Workflow sharing and troubleshooting
- AI Video Generation: Best practices and prompt libraries

### Demo

[Watch the demo video on Google Drive](https://drive.google.com/file/d/1QzsrfoT0lNwl3patwHF4m1otKwMeXr8l/view?usp=drive_link)
---

*Generated by GenAI Video Generation Pipeline v2.1 - Optimized for reproducible, brand-compliant video content creation*
