# CODEX PROMPT: Build Agentic Video Creation & Editing Platform

## PROJECT OVERVIEW
Create a complete agentic video creation and editing web application that allows users to create professional videos through natural language conversation without prompt engineering. The system should use AI agents to understand user intent, generate content, edit videos, and handle playback.

## CORE REQUIREMENTS

### 1. AGENT ARCHITECTURE
Build a multi-agent system with these components:

**Master Orchestrator Agent**
- Parse natural language requests ("Make me a marketing video for my bakery")
- Understand context from uploaded files (images, logos, brand assets)
- Break down complex requests into actionable tasks
- Coordinate between specialized agents
- Learn user preferences over time

**Content Generation Agent**
- Generate video scripts based on business type/goals
- Source and suggest stock footage, images, music
- Create custom graphics and text overlays
- Generate voiceovers using text-to-speech
- Plan video structure and pacing

**Video Editor Agent**
- Auto-sequence clips based on content analysis
- Apply transitions, effects, and filters
- Handle color correction and audio mixing
- Add text overlays and motion graphics
- Export in multiple formats and aspect ratios

**Player Agent**
- Handle video playback with custom controls
- Provide analytics and engagement tracking
- Export and sharing capabilities
- Social media optimization

### 2. TECHNICAL STACK
Use modern web technologies:
- **Frontend**: React/Next.js with TypeScript
- **Backend**: Node.js with Express or FastAPI
- **AI Integration**: OpenAI API, Anthropic Claude API
- **Video Processing**: FFmpeg, WebCodecs API
- **File Storage**: AWS S3 or similar
- **Database**: PostgreSQL or MongoDB
- **Real-time**: WebSocket for live updates

### 3. USER INTERFACE COMPONENTS

**Main Chat Interface**
```jsx
// Natural language input component
const ChatInterface = () => {
  const [messages, setMessages] = useState([]);
  const [currentProject, setCurrentProject] = useState(null);
  
  const handleUserMessage = async (message) => {
    // Process with Master Orchestrator
    const response = await processUserIntent(message);
    // Update UI based on agent response
  };
  
  return (
    <div className="chat-container">
      <MessageList messages={messages} />
      <VideoPreview project={currentProject} />
      <InputArea onMessage={handleUserMessage} />
    </div>
  );
};
```

**Asset Manager**
```jsx
// Drag-and-drop file upload and organization
const AssetManager = () => {
  const [assets, setAssets] = useState([]);
  
  const handleFileUpload = (files) => {
    // Process and categorize uploaded files
    // Extract metadata (brand colors, logos, etc.)
  };
  
  return (
    <div className="asset-manager">
      <DropZone onUpload={handleFileUpload} />
      <AssetGrid assets={assets} />
    </div>
  );
};
```

**Video Preview System**
```jsx
// Real-time video preview during creation
const VideoPreview = ({ project }) => {
  const [previewUrl, setPreviewUrl] = useState(null);
  
  useEffect(() => {
    if (project) {
      generatePreview(project).then(setPreviewUrl);
    }
  }, [project]);
  
  return (
    <div className="video-preview">
      <video src={previewUrl} controls />
      <Timeline project={project} />
    </div>
  );
};
```

### 4. AGENT IMPLEMENTATION

**Master Orchestrator Logic**
```javascript
class MasterOrchestrator {
  async processUserIntent(message, context) {
    // Analyze user intent using NLP
    const intent = await this.analyzeIntent(message);
    
    // Extract relevant context
    const projectContext = this.extractContext(context);
    
    // Determine required agents and tasks
    const tasks = this.decomposeTasks(intent, projectContext);
    
    // Coordinate agent execution
    const results = await this.executeAgents(tasks);
    
    return this.synthesizeResponse(results);
  }
  
  async analyzeIntent(message) {
    const prompt = `
    Analyze this user request for video creation:
    "${message}"
    
    Return JSON with:
    - video_type: (promotional, educational, social, etc.)
    - business_context: extracted business information
    - requirements: specific needs mentioned
    - tone: desired video tone/style
    - urgency: timeline if mentioned
    `;
    
    return await this.callAI(prompt);
  }
}
```

**Content Generation Agent**
```javascript
class ContentAgent {
  async generateScript(intent, assets) {
    const prompt = `
    Create a video script for:
    Business: ${intent.business_context}
    Video Type: ${intent.video_type}
    Tone: ${intent.tone}
    
    Available Assets: ${assets.map(a => a.type).join(', ')}
    
    Return JSON with:
    - script: array of scenes with dialogue/voiceover
    - visual_cues: what to show in each scene
    - duration: estimated duration for each scene
    - music_style: recommended music type
    `;
    
    return await this.callAI(prompt);
  }
  
  async suggestAssets(script) {
    // Analyze script and suggest stock footage/images
    // Return categorized asset suggestions
  }
}
```

**Video Editor Agent**
```javascript
class EditorAgent {
  async assembleVideo(script, assets) {
    const timeline = this.createTimeline(script);
    
    for (const scene of script.scenes) {
      const clip = await this.processScene(scene, assets);
      timeline.addClip(clip);
    }
    
    // Apply transitions and effects
    await this.applyTransitions(timeline);
    await this.mixAudio(timeline);
    
    return timeline;
  }
  
  async processScene(scene, assets) {
    // Match assets to scene requirements
    const matchedAssets = this.matchAssets(scene, assets);
    
    // Apply effects based on scene mood
    const effects = this.determineEffects(scene);
    
    return {
      assets: matchedAssets,
      effects: effects,
      duration: scene.duration
    };
  }
}
```

### 5. API ENDPOINTS

**Project Management**
```javascript
// POST /api/projects
app.post('/api/projects', async (req, res) => {
  const { userMessage, assets } = req.body;
  
  const project = await orchestrator.createProject(userMessage, assets);
  res.json(project);
});

// GET /api/projects/:id/preview
app.get('/api/projects/:id/preview', async (req, res) => {
  const preview = await editorAgent.generatePreview(req.params.id);
  res.json({ previewUrl: preview });
});

// POST /api/projects/:id/render
app.post('/api/projects/:id/render', async (req, res) => {
  const { format, quality } = req.body;
  const renderJob = await editorAgent.renderVideo(req.params.id, { format, quality });
  res.json({ jobId: renderJob.id });
});
```

**Chat Interface**
```javascript
// POST /api/chat
app.post('/api/chat', async (req, res) => {
  const { message, projectId, context } = req.body;
  
  const response = await orchestrator.processUserIntent(message, {
    projectId,
    ...context
  });
  
  res.json(response);
});
```

### 6. VIDEO PROCESSING PIPELINE

**Video Assembly**
```javascript
class VideoProcessor {
  async assembleVideo(timeline) {
    const ffmpegCommand = this.buildFFmpegCommand(timeline);
    
    // Process video using FFmpeg
    const outputPath = await this.executeFFmpeg(ffmpegCommand);
    
    // Upload to storage
    const videoUrl = await this.uploadToStorage(outputPath);
    
    return videoUrl;
  }
  
  buildFFmpegCommand(timeline) {
    let command = 'ffmpeg';
    
    // Add input files
    timeline.clips.forEach((clip, index) => {
      command += ` -i ${clip.path}`;
    });
    
    // Add filters and effects
    command += this.buildFilterGraph(timeline);
    
    // Set output parameters
    command += ' -c:v libx264 -preset medium -crf 23';
    command += ' -c:a aac -b:a 128k';
    
    return command;
  }
}
```

### 7. REAL-TIME FEATURES

**WebSocket Integration**
```javascript
// Real-time updates during video processing
const io = require('socket.io')(server);

io.on('connection', (socket) => {
  socket.on('join-project', (projectId) => {
    socket.join(projectId);
  });
  
  socket.on('chat-message', async (data) => {
    const response = await orchestrator.processUserIntent(data.message, data.context);
    socket.to(data.projectId).emit('agent-response', response);
  });
});

// Emit progress updates
class ProgressTracker {
  updateProgress(projectId, stage, progress) {
    io.to(projectId).emit('progress-update', {
      stage,
      progress,
      timestamp: new Date()
    });
  }
}
```

### 8. DEPLOYMENT & SCALING

**Docker Configuration**
```dockerfile
# Dockerfile
FROM node:18-alpine

# Install FFmpeg
RUN apk add --no-cache ffmpeg

# Copy application
COPY . /app
WORKDIR /app

# Install dependencies
RUN npm install

# Expose port
EXPOSE 3000

CMD ["npm", "start"]
```

**Environment Variables**
```bash
# .env
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_anthropic_key
AWS_ACCESS_KEY_ID=your_aws_key
AWS_SECRET_ACCESS_KEY=your_aws_secret
DATABASE_URL=postgresql://user:pass@host:5432/db
REDIS_URL=redis://localhost:6379
```

## IMPLEMENTATION CHECKLIST

### Phase 1: Core Foundation
- [ ] Set up project structure with React/Node.js
- [ ] Implement Master Orchestrator Agent
- [ ] Create basic chat interface
- [ ] Build file upload system
- [ ] Integrate OpenAI/Anthropic APIs
- [ ] Set up database schemas

### Phase 2: Agent Intelligence
- [ ] Implement Content Generation Agent
- [ ] Build Video Editor Agent logic
- [ ] Create video processing pipeline
- [ ] Add real-time preview system
- [ ] Implement user preference learning

### Phase 3: Advanced Features
- [ ] Add Player Agent with analytics
- [ ] Implement export/sharing features
- [ ] Build collaborative features
- [ ] Add social media integrations
- [ ] Optimize performance and scaling

### Phase 4: Polish & Deploy
- [ ] Comprehensive testing
- [ ] Performance optimization
- [ ] Security hardening
- [ ] Documentation
- [ ] Production deployment

## EXAMPLE USAGE FLOW

1. User types: "Create a promotional video for my coffee shop's grand opening"
2. System analyzes intent and asks for assets
3. User uploads logo, shop photos, and mentions date
4. Content Agent generates script and suggests music
5. Editor Agent assembles video with transitions and effects
6. User reviews preview and requests "make it more energetic"
7. System adjusts pacing, music, and effects automatically
8. Final video is rendered and ready for download/sharing

## DELIVERABLES

Provide a complete, working web application with:
- Fully functional agent system
- Modern, responsive UI
- Real-time video processing
- Complete API documentation
- Deployment instructions
- Example usage scenarios

Build this as a production-ready application with proper error handling, security measures, and scalability considerations.
