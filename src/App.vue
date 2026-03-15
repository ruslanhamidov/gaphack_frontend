<template>
  <div class="app" :class="{ 'nav-open': mobileNavOpen }">
    <div class="scanlines" aria-hidden="true"></div>

    <header class="nav">
      <div class="nav-inner">
        <button class="logo" @click="go('landing')">
          <span class="logo-bracket">[</span>GAP<span class="logo-accent">HACK</span><span class="logo-bracket">]</span>
        </button>
        <nav class="nav-links">
          <button @click="go('tasks')" :class="{ active: view === 'tasks' }">Tasks</button>
          <button @click="go('leaderboard')" :class="{ active: view === 'leaderboard' }">Leaderboard</button>
          <button v-if="currentUser" @click="go('dashboard')" :class="{ active: view === 'dashboard' }">Dashboard</button>
          <DiscordButton />
        </nav>
        <div class="nav-actions">
          <template v-if="!currentUser">
            <button class="btn-ghost" @click="go('login')">Sign in</button>
            <button class="btn-primary btn-lg" @click="currentUser ? go('tasks') : go('register')">Get Started →</button>
          </template>
          <template v-else>
            <button class="user-pill" @click="viewUserProfile(currentUser.username)">
              <span class="user-dot"></span>{{ currentUser.username }}
            </button>
            <button class="btn-ghost" @click="logout">Logout</button>
          </template>
        </div>
      </div>
    </header>

    <main class="main">

      <!-- ===== LANDING ===== -->
      <section v-if="view === 'landing'" class="landing">
        <div class="hero">
          <div class="hero-tag">// cybersecurity_gap_analysis_platform</div>
          <h1 class="hero-title">
            Hack the <span class="accent">Gap.</span><br>
            Build the <span class="accent">Portfolio.</span>
          </h1>
          <p class="hero-sub">
            GapHack connects companies with cybersecurity researchers to discover vulnerabilities,
            build real-world portfolios, and crowdsource security gap analysis — without expensive bug bounties.
          </p>
          <div class="hero-cta">
            <button class="btn-primary btn-lg" @click="handleGetStarted">Get Started →</button>
            <button class="btn-outline btn-lg" @click="go('tasks')">Browse Tasks</button>
            <DiscordButton variant="lg" />
          </div>
          <div class="hero-stats">
            <div class="stat"><span class="stat-num">{{ tasks.length || 247 }}</span><span class="stat-label">Active Tasks</span></div>
            <div class="stat-div"></div>
            <div class="stat"><span class="stat-num">{{ leaderboard.length || 0 }}</span><span class="stat-label">Researchers</span></div>
            <div class="stat-div"></div>
            <div class="stat"><span class="stat-num">94</span><span class="stat-label">Companies</span></div>
            <div class="stat-div"></div>
            <div class="stat"><span class="stat-num">5,310</span><span class="stat-label">Findings</span></div>
          </div>
        </div>
        <div class="features">
          <div class="feature-card">
            <div class="feature-icon">⚡</div>
            <h3>For Researchers</h3>
            <p>Browse real-world security challenges, submit findings, earn badges, and build a public portfolio that proves your skills.</p>
          </div>
          <div class="feature-card feature-card--accent">
            <div class="feature-icon">🏢</div>
            <h3>For Companies</h3>
            <p>Post scoped security tasks, get community-driven analysis, and reward impactful findings — all without enterprise-level costs.</p>
          </div>
          <div class="feature-card">
            <div class="feature-icon">🏆</div>
            <h3>Recognition System</h3>
            <p>Earn reputation points, collect achievement badges, and climb the leaderboard as your contributions are verified and upvoted.</p>
          </div>
        </div>
        <div class="live-feed">
          <div class="section-label">// recent_activity</div>
          <div class="feed-list">
            <div class="feed-item" v-for="item in activityFeed" :key="item.id">
              <span class="feed-time">{{ item.time }}</span>
              <span class="feed-user">{{ item.user }}</span>
              <span class="feed-action">{{ item.action }}</span>
              <span class="feed-target">{{ item.target }}</span>
            </div>
          </div>
        </div>
      </section>

      <!-- ===== LOGIN ===== -->
      <section v-else-if="view === 'login'" class="auth-page">
        <div class="auth-card">
          <div class="auth-tag">// user_authentication</div>
          <h2>Sign In</h2>
          <p class="auth-sub">Access your GapHack account</p>
          <div class="form-group">
            <label>Username</label>
            <input v-model="loginForm.username" type="text" placeholder="your_username" class="input" @keyup.enter="doLogin" />
          </div>
          <div class="form-group">
            <label>Password</label>
            <input v-model="loginForm.password" type="password" placeholder="••••••••" class="input" @keyup.enter="doLogin" />
          </div>
          <div v-if="authError" class="auth-error">{{ authError }}</div>
          <button class="btn-primary btn-full" @click="doLogin" :disabled="loginLoading">
            {{ loginLoading ? 'Signing in...' : 'Sign In →' }}
          </button>
          <p class="auth-switch">No account? <button @click="go('register')">Register here</button></p>
          <div class="demo-logins">
            <div class="demo-label">// demo_accounts</div>
            <button class="demo-btn" @click="demoLogin('ghost_0x1', 'demo123')">🔍 Login as Researcher</button>
            <button class="demo-btn" @click="demoLogin('acme_corp', 'demo123')">🏢 Login as Company</button>
          </div>
        </div>
      </section>

      <!-- ===== REGISTER ===== -->
      <section v-else-if="view === 'register'" class="auth-page">
        <div class="auth-card">
          <div class="auth-tag">// create_account</div>
          <h2>Join GapHack</h2>
          <p class="auth-sub">Start building your security portfolio</p>
          <div class="form-group">
            <label>Username</label>
            <input v-model="registerForm.username" type="text" placeholder="ghost_researcher" class="input" />
          </div>
          <div class="form-group">
            <label>Password</label>
            <input v-model="registerForm.password" type="password" placeholder="••••••••" class="input" />
          </div>
          <div class="form-group">
            <label>Email</label>
            <input v-model="registerForm.email" type="email" placeholder="you@domain.com" class="input" />
          </div>
          <div class="form-group">
            <label>I am a...</label>
            <div class="role-picker">
              <button :class="['role-btn', { active: registerForm.role === 'researcher' }]" @click="registerForm.role = 'researcher'">
                <span>🔍</span> Researcher
              </button>
              <button :class="['role-btn', { active: registerForm.role === 'company' }]" @click="registerForm.role = 'company'">
                <span>🏢</span> Company
              </button>
            </div>
          </div>
          <div v-if="authError" class="auth-error">{{ authError }}</div>
          <button class="btn-primary btn-full" @click="doRegister" :disabled="registerLoading">
            {{ registerLoading ? 'Creating...' : 'Create Account →' }}
          </button>
          <p class="auth-switch">Have an account? <button @click="go('login')">Sign in</button></p>
        </div>
      </section>

      <!-- ===== TASKS ===== -->
      <section v-else-if="view === 'tasks'" class="tasks-page">
        <div class="page-header">
          <div>
            <div class="section-label">// security_tasks</div>
            <h2>Open Challenges</h2>
          </div>
          <button v-if="currentUser?.role === 'company'" class="btn-primary" @click="go('post-task')">+ Post Task</button>
        </div>
        <div class="filters">
          <button v-for="cat in categories" :key="cat"
            :class="['filter-btn', { active: activeCategory === cat }]"
            @click="activeCategory = cat">{{ cat }}</button>
        </div>
        <p v-if="tasksLoading" class="loading-text">// loading tasks from backend...</p>
        <p v-else-if="tasksError" class="error-text">{{ tasksError }}</p>
        <div v-else class="tasks-grid">
          <div class="task-card" v-for="task in filteredTasks" :key="task.id" @click="openTask(task)">
            <div class="task-card-header">
              <span :class="['difficulty-badge', task.difficulty.toLowerCase()]">{{ task.difficulty }}</span>
              <span class="task-pts">+{{ task.points }} pts</span>
            </div>
            <h3 class="task-title">{{ task.title }}</h3>
            <p class="task-desc">{{ task.description }}</p>
            <div class="task-meta">
              <span class="task-company">{{ task.company }}</span>
              <div class="task-tags">
                <span class="tag" v-for="tag in task.tags" :key="tag">{{ tag }}</span>
              </div>
            </div>
            <div class="task-footer">
              <span class="task-stat">💬 {{ task.submissions }} submissions</span>
              <span class="task-stat">⬆ {{ task.upvotes }} upvotes</span>
            </div>
          </div>
        </div>
      </section>

      <!-- ===== TASK DETAIL ===== -->
      <section v-else-if="view === 'task-detail' && selectedTask" class="task-detail">
        <button class="back-btn" @click="go('tasks')">← Back to Tasks</button>
        <div class="task-detail-header">
          <div class="task-detail-meta">
            <span :class="['difficulty-badge', selectedTask.difficulty.toLowerCase()]">{{ selectedTask.difficulty }}</span>
            <span class="task-company">{{ selectedTask.company }}</span>
            <span class="task-pts-lg">+{{ selectedTask.points }} pts</span>
          </div>
          <h2>{{ selectedTask.title }}</h2>
          <p>{{ selectedTask.description }}</p>
          <div class="task-detail-tags">
            <span class="tag" v-for="tag in selectedTask.tags" :key="tag">{{ tag }}</span>
          </div>
        </div>
        <div class="task-layout">
          <div class="thread-section">
            <div class="section-label">// submissions_thread ({{ taskSubmissions.length }})</div>
            <div class="submit-box" v-if="currentUser && currentUser.role !== 'company'">
              <div class="submit-header">
                <span class="user-dot"></span>
                <span>{{ currentUser.username }}</span>
              </div>
              <textarea v-model="newSubmission" class="input textarea" placeholder="Describe the vulnerability or security gap you found. Include reproduction steps, impact, and suggested mitigation..."></textarea>
              <div class="submit-actions">
                <button class="btn-primary" @click="submitFinding" :disabled="submitLoading">
                  {{ submitLoading ? 'Submitting...' : 'Submit Finding →' }}
                </button>
              </div>
            </div>
            <div v-else-if="!currentUser" class="login-prompt">
              <button @click="go('login')">Sign in to submit a finding</button>
            </div>
            <p v-if="submissionsLoading" class="loading-text">// loading submissions...</p>
            <div v-else class="thread">
              <div class="thread-post" v-for="post in taskSubmissions" :key="post.id">
                <div class="post-avatar">{{ post.user[0].toUpperCase() }}</div>
                <div class="post-body">
                  <div class="post-header">
                    <button class="post-user" @click="viewUserProfile(post.user)">{{ post.user }}</button>
                    <span class="post-time">{{ post.time }}</span>
                    <span v-if="post.status" :class="['post-status', post.status]">{{ post.status }}</span>
                  </div>
                  <p class="post-content">{{ post.content }}</p>
                  <div v-if="post.code" class="post-code"><pre>{{ post.code }}</pre></div>
                  <div class="post-actions">
                    <button class="vote-btn" @click="upvoteSubmission(post)">⬆ {{ post.votes }}</button>
                    <button class="reply-btn">💬 Reply</button>
                    <button v-if="currentUser?.role === 'company'" class="verify-btn" @click="rewardSubmission(post)">✓ Reward</button>
                  </div>
                </div>
              </div>
            </div>
          </div>
          <div class="task-sidebar">
            <div class="sidebar-card">
              <div class="section-label">// task_info</div>
              <div class="info-row"><span>Category</span><span>{{ selectedTask.category }}</span></div>
              <div class="info-row"><span>Difficulty</span><span :class="['difficulty-text', selectedTask.difficulty.toLowerCase()]">{{ selectedTask.difficulty }}</span></div>
              <div class="info-row"><span>Reward</span><span class="accent-text">{{ selectedTask.points }} pts</span></div>
              <div class="info-row"><span>Submissions</span><span>{{ taskSubmissions.length }}</span></div>
              <div class="info-row"><span>Posted by</span><span>{{ selectedTask.company }}</span></div>
            </div>
            <div class="sidebar-card">
              <div class="section-label">// top_contributors</div>
              <div class="contributor" v-for="c in topContributors" :key="c.name">
                <div class="contrib-avatar">{{ c.name[0] }}</div>
                <span>{{ c.name }}</span>
                <span class="contrib-pts">{{ c.pts }} pts</span>
              </div>
              <p v-if="!topContributors.length" class="loading-text" style="margin:0">// no submissions yet</p>
            </div>
          </div>
        </div>
      </section>

      <!-- ===== POST TASK ===== -->
      <section v-else-if="view === 'post-task'" class="auth-page">
        <div class="auth-card auth-card--wide">
          <div class="auth-tag">// post_security_task</div>
          <h2>Create a Task</h2>
          <div class="form-group">
            <label>Title</label>
            <input v-model="taskForm.title" type="text" placeholder="e.g. SQL Injection in login endpoint" class="input" />
          </div>
          <div class="form-group">
            <label>Description</label>
            <textarea v-model="taskForm.description" class="input textarea" placeholder="Describe the security scenario, scope, and what you're looking for..."></textarea>
          </div>
          <div class="form-row">
            <div class="form-group">
              <label>Category</label>
              <select v-model="taskForm.category" class="input">
                <option v-for="cat in categories.slice(1)" :key="cat">{{ cat }}</option>
              </select>
            </div>
            <div class="form-group">
              <label>Difficulty</label>
              <select v-model="taskForm.difficulty" class="input">
                <option>Easy</option><option>Medium</option><option>Hard</option><option>Critical</option>
              </select>
            </div>
          </div>
          <div class="form-group">
            <label>Tags (comma separated)</label>
            <input v-model="taskForm.tags" type="text" placeholder="XSS, CSRF, authentication" class="input" />
          </div>
          <div class="form-group">
            <label>Reward Points</label>
            <input v-model="taskForm.points" type="number" placeholder="500" class="input" />
          </div>
          <div v-if="postTaskError" class="auth-error">{{ postTaskError }}</div>
          <div v-if="postTaskSuccess" class="auth-success">{{ postTaskSuccess }}</div>
          <button class="btn-primary btn-full" @click="postTask" :disabled="postTaskLoading">
            {{ postTaskLoading ? 'Publishing...' : 'Publish Task →' }}
          </button>
        </div>
      </section>

      <!-- ===== PROFILE ===== -->
      <section v-else-if="view === 'profile'" class="profile-page">
        <p v-if="profileLoading" class="loading-text">// loading profile...</p>
        <template v-else>
          <div class="profile-header">
            <div class="profile-avatar">{{ (profileUser || currentUser)?.username[0]?.toUpperCase() }}</div>
            <div class="profile-info">
              <div class="section-label">// user_profile</div>
              <h2>{{ (profileUser || currentUser)?.username }}</h2>
              <p class="profile-bio">{{ (profileUser || currentUser)?.bio || 'Cybersecurity researcher & vulnerability hunter' }}</p>
              <div class="profile-stats">
                <template v-if="(profileUser || currentUser)?.role !== 'company'">
                  <div class="pstat"><span class="pstat-num">{{ (profileUser || currentUser)?.reputation || 0 }}</span><span>Reputation</span></div>
                  <div class="pstat"><span class="pstat-num">{{ (profileUser || currentUser)?.findings || 0 }}</span><span>Findings</span></div>
                  <div class="pstat"><span class="pstat-num">{{ (profileUser || currentUser)?.badges?.length || 0 }}</span><span>Badges</span></div>
                </template>
                <template v-else>
                  <div class="pstat"><span class="pstat-num">{{ companyTasks.length }}</span><span>Tasks Posted</span></div>
                  <div class="pstat"><span class="pstat-num">{{ companyTasks.reduce((a, t) => a + t.submissions, 0) }}</span><span>Submissions</span></div>
                  <div class="pstat"><span class="pstat-num">{{ (profileUser || currentUser)?.reputation || 0 }}</span><span>Reputation</span></div>
                </template>
              </div>
            </div>
          </div>

          <div class="profile-body">
            <!-- RESEARCHER -->
            <div class="profile-layout" v-if="(profileUser || currentUser)?.role !== 'company'">
              <div class="profile-main">
                <div class="section-label">// recent_submissions</div>
                <div class="submission-list">
                  <div v-if="!userSubmissions.length" class="loading-text">// no submissions yet</div>
                  <div class="submission-item" v-for="s in userSubmissions" :key="s.id">
                    <div class="sub-task">{{ s.task }}</div>
                    <div class="sub-content">{{ s.content }}</div>
                    <div class="sub-meta">
                      <span :class="['post-status', s.status]">{{ s.status }}</span>
                      <span>⬆ {{ s.votes }}</span>
                      <span>{{ s.time }}</span>
                    </div>
                  </div>
                </div>
              </div>
              <div class="profile-side">
                <div class="sidebar-card">
                  <div class="section-label">// badges</div>
                  <div v-if="!(profileUser || currentUser)?.badges?.length" class="loading-text" style="margin:0">// no badges yet</div>
                  <div class="badge-grid">
                    <div class="badge-item" v-for="badge in (profileUser || currentUser)?.badges || []" :key="badge.name || badge.badge_type" :title="badge.desc">
                      <span class="badge-icon">{{ badge.icon || '🔍' }}</span>
                      <span class="badge-name">{{ badge.name || badge.badge_type }}</span>
                    </div>
                  </div>
                </div>
                <div class="sidebar-card">
                  <div class="section-label">// skills</div>
                  <div class="skill-bar" v-for="skill in userSkills" :key="skill.name">
                    <div class="skill-label"><span>{{ skill.name }}</span><span>{{ skill.level }}%</span></div>
                    <div class="skill-track"><div class="skill-fill" :style="{ width: skill.level + '%' }"></div></div>
                  </div>
                </div>
              </div>
            </div>

            <!-- COMPANY -->
            <div class="profile-layout" v-else>
              <div class="profile-main">
                <div class="section-label">// posted_tasks</div>
                <div class="submission-list">
                  <div v-if="!companyTasks.length" class="loading-text">// no tasks posted yet</div>
                  <div class="submission-item" v-for="t in companyTasks" :key="t.id" @click="openTask(t)" style="cursor:pointer">
                    <div class="sub-task">{{ t.category }}</div>
                    <div class="sub-content">{{ t.title }}</div>
                    <div class="sub-meta">
                      <span :class="['difficulty-badge', t.difficulty.toLowerCase()]">{{ t.difficulty }}</span>
                      <span>💬 {{ t.submissions }} submissions</span>
                      <span class="accent-text">+{{ t.points }} pts</span>
                    </div>
                  </div>
                </div>
              </div>
              <div class="profile-side">
                <div class="sidebar-card">
                  <div class="section-label">// company_stats</div>
                  <div class="info-row"><span>Tasks Posted</span><span class="accent-text">{{ companyTasks.length }}</span></div>
                  <div class="info-row"><span>Total Submissions</span><span>{{ companyTasks.reduce((a, t) => a + t.submissions, 0) }}</span></div>
                  <div class="info-row"><span>Role</span><span>Company</span></div>
                  <div class="info-row"><span>Reputation</span><span class="accent-text">{{ (profileUser || currentUser)?.reputation || 0 }}</span></div>
                </div>
                <div class="sidebar-card">
                  <div class="section-label">// about</div>
                  <p style="font-size:0.82rem; color:var(--text2); line-height:1.6">
                    {{ (profileUser || currentUser)?.bio || 'Security-focused company posting challenges for the community.' }}
                  </p>
                </div>
              </div>
            </div>
          </div>
        </template>
      </section>

      <!-- ===== LEADERBOARD ===== -->
      <section v-else-if="view === 'leaderboard'" class="leaderboard-page">
        <div class="page-header">
          <div>
            <div class="section-label">// reputation_leaderboard</div>
            <h2>Top Researchers</h2>
          </div>
        </div>
        <p v-if="leaderboardLoading" class="loading-text">// loading leaderboard...</p>
        <p v-else-if="leaderboardError" class="error-text">{{ leaderboardError }}</p>
        <template v-else-if="leaderboard.length >= 3">
          <div class="podium">
            <div class="podium-slot podium-2" @click="viewUserProfile(leaderboard[1].username)">
              <div class="podium-avatar">{{ leaderboard[1].username[0] }}</div>
              <div class="podium-rank">#2</div>
              <div class="podium-name">{{ leaderboard[1].username }}</div>
              <div class="podium-pts">{{ leaderboard[1].reputation }} pts</div>
            </div>
            <div class="podium-slot podium-1" @click="viewUserProfile(leaderboard[0].username)">
              <div class="podium-crown">👑</div>
              <div class="podium-avatar podium-avatar--lg">{{ leaderboard[0].username[0] }}</div>
              <div class="podium-rank">#1</div>
              <div class="podium-name">{{ leaderboard[0].username }}</div>
              <div class="podium-pts">{{ leaderboard[0].reputation }} pts</div>
            </div>
            <div class="podium-slot podium-3" @click="viewUserProfile(leaderboard[2].username)">
              <div class="podium-avatar">{{ leaderboard[2].username[0] }}</div>
              <div class="podium-rank">#3</div>
              <div class="podium-name">{{ leaderboard[2].username }}</div>
              <div class="podium-pts">{{ leaderboard[2].reputation }} pts</div>
            </div>
          </div>
          <div class="lb-table">
            <div class="lb-row lb-header"><span>Rank</span><span>Researcher</span><span>Reputation</span><span>Badges</span></div>
            <div class="lb-row" v-for="(user, i) in leaderboard" :key="user.username" @click="viewUserProfile(user.username)">
              <span class="lb-rank" :class="{ 'lb-top': i < 3 }">#{{ i + 1 }}</span>
              <span class="lb-user"><div class="lb-avatar">{{ user.username[0] }}</div>{{ user.username }}</span>
              <span class="lb-pts">{{ user.reputation }}</span>
              <span class="lb-badges">{{ user.badges }} 🏅</span>
            </div>
          </div>
        </template>
        <template v-else-if="leaderboard.length > 0">
          <div class="lb-table">
            <div class="lb-row lb-header"><span>Rank</span><span>Researcher</span><span>Reputation</span><span>Badges</span></div>
            <div class="lb-row" v-for="(user, i) in leaderboard" :key="user.username" @click="viewUserProfile(user.username)">
              <span class="lb-rank" :class="{ 'lb-top': i < 3 }">#{{ i + 1 }}</span>
              <span class="lb-user"><div class="lb-avatar">{{ user.username[0] }}</div>{{ user.username }}</span>
              <span class="lb-pts">{{ user.reputation }}</span>
              <span class="lb-badges">{{ user.badges }} 🏅</span>
            </div>
          </div>
        </template>
        <p v-else class="loading-text">// no users yet — register some accounts first</p>
      </section>

      <!-- ===== DASHBOARD ===== -->
      <section v-else-if="view === 'dashboard'" class="dashboard">
        <div class="section-label">// {{ currentUser?.role }}_dashboard</div>
        <h2>Welcome back, {{ currentUser?.username }}</h2>
        <template v-if="currentUser?.role !== 'company'">
          <div class="dash-grid">
            <div class="dash-stat-card"><div class="dash-stat-label">Reputation</div><div class="dash-stat-num accent-text">{{ currentUser.reputation }}</div></div>
            <div class="dash-stat-card"><div class="dash-stat-label">Submissions</div><div class="dash-stat-num">{{ userSubmissions.length }}</div></div>
            <div class="dash-stat-card"><div class="dash-stat-label">Badges</div><div class="dash-stat-num">{{ currentUser.badges?.length || 0 }}</div></div>
            <div class="dash-stat-card"><div class="dash-stat-label">Open Tasks</div><div class="dash-stat-num">{{ tasks.length }}</div></div>
          </div>
          <div class="dash-section">
            <div class="section-label">// recommended_tasks</div>
            <div class="tasks-grid tasks-grid--sm">
              <div class="task-card" v-for="task in tasks.slice(0,3)" :key="task.id" @click="openTask(task)">
                <div class="task-card-header">
                  <span :class="['difficulty-badge', task.difficulty.toLowerCase()]">{{ task.difficulty }}</span>
                  <span class="task-pts">+{{ task.points }} pts</span>
                </div>
                <h3 class="task-title">{{ task.title }}</h3>
                <p class="task-desc">{{ task.description }}</p>
              </div>
            </div>
          </div>
        </template>
        <template v-else>
          <div class="dash-grid">
            <div class="dash-stat-card"><div class="dash-stat-label">Tasks Posted</div><div class="dash-stat-num accent-text">{{ myTasks.length }}</div></div>
            <div class="dash-stat-card"><div class="dash-stat-label">Total Submissions</div><div class="dash-stat-num">{{ myTasks.reduce((a, t) => a + t.submissions, 0) }}</div></div>
            <div class="dash-stat-card"><div class="dash-stat-label">Researchers</div><div class="dash-stat-num">{{ leaderboard.length }}</div></div>
          </div>
          <div class="dash-section">
            <div class="dash-section-header">
              <div class="section-label">// your_tasks</div>
              <button class="btn-primary" @click="go('post-task')">+ Post Task</button>
            </div>
            <div v-if="!myTasks.length" class="loading-text">// no tasks posted yet</div>
            <div class="tasks-grid tasks-grid--sm">
              <div class="task-card" v-for="task in myTasks" :key="task.id" @click="openTask(task)">
                <div class="task-card-header">
                  <span :class="['difficulty-badge', task.difficulty.toLowerCase()]">{{ task.difficulty }}</span>
                  <span class="task-pts">{{ task.submissions }} subs</span>
                </div>
                <h3 class="task-title">{{ task.title }}</h3>
                <div class="task-footer"><span class="task-stat">+{{ task.points }} pts</span></div>
              </div>
            </div>
          </div>
          <div class="dash-section">
            <div class="section-label">// award_badge</div>
            <div class="badge-award-form">
              <select v-model="badgeForm.user" class="input input--sm">
                <option value="">Select researcher...</option>
                <option v-for="u in leaderboard" :key="u.username" :value="u.username">{{ u.username }}</option>
              </select>
              <select v-model="badgeForm.badge" class="input input--sm">
                <option value="">Select badge...</option>
                <option v-for="b in availableBadges" :key="b.name" :value="b.name">{{ b.icon }} {{ b.name }}</option>
              </select>
              <button class="btn-primary" @click="awardBadge">Award →</button>
            </div>
            <div v-if="badgeSuccess" class="auth-success">{{ badgeSuccess }}</div>
          </div>
        </template>
      </section>

    </main>

    <footer class="footer">
      <div class="footer-inner">
        <span class="logo-small">[GAPHACK]</span>
        <span>// built_for_hackathon — cybersecurity_gap_analysis</span>
        <span>qapaq · 2026</span>
      </div>
    </footer>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, watch } from 'vue'
import DiscordButton from './components/DiscordButton.vue'

const API_BASE = import.meta.env.VITE_API_BASE_URL || 'http://localhost:8000'

// ─── Persisted state ─────────────────────────────────────
const savedUser = localStorage.getItem('gaphack_user')
const savedSubmissions = localStorage.getItem('gaphack_submissions')
const currentUser = ref(savedUser ? JSON.parse(savedUser) : null)
const userSubmissions = ref(savedSubmissions ? JSON.parse(savedSubmissions) : [])

function handleGetStarted() {
  if (currentUser.value) {
    go('tasks')
  } else {
    go('register')
  }
}

watch(currentUser, (val) => {
  if (val) localStorage.setItem('gaphack_user', JSON.stringify(val))
  else localStorage.removeItem('gaphack_user')
}, { deep: true })

watch(userSubmissions, (val) => {
  if (val?.length) localStorage.setItem('gaphack_submissions', JSON.stringify(val))
  else localStorage.removeItem('gaphack_submissions')
}, { deep: true })

// ─── State ───────────────────────────────────────────────
const view = ref('landing')
const profileUser = ref(null)
const selectedTask = ref(null)
const authError = ref('')
const badgeSuccess = ref('')
const mobileNavOpen = ref(false)
const newSubmission = ref('')
const activeCategory = ref('All')
const postTaskError = ref('')
const postTaskSuccess = ref('')

const loginLoading = ref(false)
const registerLoading = ref(false)
const tasksLoading = ref(false)
const tasksError = ref('')
const leaderboardLoading = ref(false)
const leaderboardError = ref('')
const submissionsLoading = ref(false)
const submitLoading = ref(false)
const postTaskLoading = ref(false)
const profileLoading = ref(false)

const loginForm = ref({ username: '', password: '' })
const registerForm = ref({ username: '', email: '', password: '', role: 'researcher' })
const taskForm = ref({ title: '', description: '', category: 'Authentication', difficulty: 'Medium', tags: '', points: 500 })
const badgeForm = ref({ user: '', badge: '' })

// ─── Static data ─────────────────────────────────────────
const categories = ['All', 'Authentication', 'Network Security', 'Cloud Security', 'Web Application', 'API Security', 'Social Engineering']

const availableBadges = [
  { name: 'Vulnerability Finder', icon: '🔍', desc: 'Found a confirmed vulnerability' },
  { name: 'Security Analyst', icon: '🛡️', desc: 'Demonstrated deep analytical skills' },
  { name: 'Critical Gap Hunter', icon: '⚡', desc: 'Discovered a critical security gap' },
  { name: 'Bug Slayer', icon: '🐛', desc: 'Resolved multiple bugs in sequence' },
  { name: 'Zero Day Scout', icon: '🎯', desc: 'Reported a zero-day vulnerability' },
]

const badgeIconMap = {
  'Vulnerability Finder': '🔍',
  'Security Analyst': '🛡️',
  'Critical Gap Hunter': '⚡',
  'Bug Slayer': '🐛',
  'Zero Day Scout': '🎯',
  'Security Finding': '🔐',
}

const activityFeed = ref([
  { id: 1, time: '2m ago', user: 'security_researcher', action: 'submitted a finding on', target: 'Find XSS vulnerability in login form' },
  { id: 2, time: '34m ago', user: 'cyber_ninja', action: 'submitted on', target: 'Detect SQL injection in API endpoint' },
  { id: 3, time: '1h ago', user: 'acme_corp', action: 'posted new task', target: 'Find XSS vulnerability in login form' },
])

const userSkills = ref([
  { name: 'Web Application Security', level: 87 },
  { name: 'Network Penetration', level: 72 },
  { name: 'Cloud Security', level: 65 },
  { name: 'Reverse Engineering', level: 50 },
  { name: 'Social Engineering', level: 40 },
])

const topContributors = ref([])
const tasks = ref([])
const leaderboard = ref([])
const taskSubmissions = ref([])

// ─── Computed ─────────────────────────────────────────────
const filteredTasks = computed(() =>
  activeCategory.value === 'All'
    ? tasks.value
    : tasks.value.filter(t => t.category === activeCategory.value)
)

const myTasks = computed(() =>
  tasks.value.filter(t => t.company === currentUser.value?.username)
)

const companyTasks = computed(() => {
  const username = (profileUser.value || currentUser.value)?.username
  return tasks.value.filter(t => t.company === username)
})

// ─── API helper ──────────────────────────────────────────
async function apiFetch(path, options = {}) {
  const res = await fetch(`${API_BASE}${path}`, {
    headers: { 'Content-Type': 'application/json' },
    ...options,
  })
  if (!res.ok) {
    const err = await res.json().catch(() => ({}))
    throw new Error(err.detail || `Request failed (${res.status})`)
  }
  return res.json()
}

// ─── Loaders ─────────────────────────────────────────────
async function loadTasks() {
  tasksLoading.value = true
  tasksError.value = ''
  try {
    const data = await apiFetch('/tasks')
    tasks.value = data.map(t => ({
      id: t.id,
      title: t.title,
      description: t.description || '',
      category: t.category || 'Web Application',
      difficulty: t.difficulty,
      tags: Array.isArray(t.tags) ? t.tags : (t.tags ? t.tags.split(',').map(s => s.trim()).filter(Boolean) : []),
      company: t.company || 'Unknown',
      company_id: t.company_id,
      points: t.reward_points,
      submissions: t.submission_count || 0,
      upvotes: 0,
    }))
  } catch (e) {
    tasksError.value = `// error connecting to backend: ${e.message}`
  } finally {
    tasksLoading.value = false
  }
}

async function loadLeaderboard() {
  leaderboardLoading.value = true
  leaderboardError.value = ''
  try {
    const data = await apiFetch('/leaderboard')
    leaderboard.value = data.map(u => ({
      username: u.username,
      reputation: u.reputation,
      badges: u.badges,
    }))
  } catch (e) {
    leaderboardError.value = `// error: ${e.message}`
  } finally {
    leaderboardLoading.value = false
  }
}

async function loadTaskSubmissions(taskId) {
  submissionsLoading.value = true
  taskSubmissions.value = []
  try {
    const data = await apiFetch(`/tasks/${taskId}`)
    taskSubmissions.value = (data.submissions || []).map(s => ({
      id: s.id,
      user: s.user?.username || 'unknown',
      time: new Date(s.created_at).toLocaleString(),
      votes: s.upvotes || 0,
      status: s.status === 'pending' ? null : s.status,
      content: s.content,
      code: null,
    }))
    const contribMap = {}
    taskSubmissions.value.forEach(s => {
      contribMap[s.user] = (contribMap[s.user] || 0) + s.votes
    })
    topContributors.value = Object.entries(contribMap)
      .sort((a, b) => b[1] - a[1]).slice(0, 3)
      .map(([name, pts]) => ({ name, pts }))
  } catch (e) {
    console.error('Failed to load submissions', e)
  } finally {
    submissionsLoading.value = false
  }
}

async function loadUserProfile(userId) {
  profileLoading.value = true
  try {
    const data = await apiFetch(`/users/${userId}`)
    userSubmissions.value = (data.submissions || []).map(s => ({
      id: s.id,
      task: s.task_title || `Task #${s.task_id}`,
      content: s.content,
      status: s.status,
      votes: s.upvotes || 0,
      time: new Date(s.created_at).toLocaleString(),
    }))
    return {
      id: data.id,
      username: data.username,
      role: data.role,
      reputation: data.reputation,
      bio: data.bio || 'Cybersecurity researcher & vulnerability hunter',
      badges: (data.badges || []).map(b => ({
        name: b.badge_type,
        icon: badgeIconMap[b.badge_type] || '🔍',
        desc: '',
      })),
      findings: (data.submissions || []).length,
    }
  } catch (e) {
    console.error('Failed to load profile', e)
    return null
  } finally {
    profileLoading.value = false
  }
}

// ─── Auth ────────────────────────────────────────────────
async function doLogin() {
  if (!loginForm.value.username) { authError.value = 'Enter your username'; return }
  if (!loginForm.value.password) { authError.value = 'Enter your password'; return }
  loginLoading.value = true
  authError.value = ''
  try {
    const data = await apiFetch('/login', {
      method: 'POST',
      body: JSON.stringify({ username: loginForm.value.username, password: loginForm.value.password }),
    })
    currentUser.value = {
      id: data.id, username: data.username, role: data.role,
      reputation: data.reputation, bio: data.bio || '',
      badges: (data.badges || []).map(b => ({ name: b.badge_type, icon: badgeIconMap[b.badge_type] || '🔍', desc: '' })),
      findings: (data.submissions || []).length,
    }
    userSubmissions.value = (data.submissions || []).map(s => ({
      id: s.id, task: s.task_title || `Task #${s.task_id}`,
      content: s.content, status: s.status,
      votes: s.upvotes || 0, time: new Date(s.created_at).toLocaleString(),
    }))
    go('dashboard')
  } catch (e) {
    authError.value = e.message.includes('401') ? 'Incorrect password' : e.message.includes('404') ? 'User not found' : `Cannot connect to backend: ${e.message}`
  } finally {
    loginLoading.value = false
  }
}

async function demoLogin(username, password) {
  loginForm.value.username = username
  loginForm.value.password = password
  await doLogin()
}

async function doRegister() {
  const f = registerForm.value
  if (!f.username) { authError.value = 'Username is required'; return }
  if (!f.password) { authError.value = 'Password is required'; return }
  if (!f.email || !f.email.includes('@') || !f.email.includes('.')) {
    authError.value = 'Please enter a valid email address'; return
  }
  registerLoading.value = true
  authError.value = ''
  try {
    const data = await apiFetch('/users', {
      method: 'POST',
      body: JSON.stringify({
      username: f.username,
      role: f.role === 'company' ? 'company' : 'user',
      password: f.password,
      email: f.email,      // ← add this
      reputation: 0,
    }),
    })
    currentUser.value = { id: data.id, username: data.username, role: data.role, reputation: 0, bio: '', badges: [], findings: 0 }
    userSubmissions.value = []
    go('dashboard')
  } catch (e) {
    authError.value = e.message
  } finally {
    registerLoading.value = false
  }
}

function logout() {
  currentUser.value = null
  localStorage.removeItem('gaphack_user')
  localStorage.removeItem('gaphack_submissions')
  userSubmissions.value = []
  go('landing')
}

// ─── Navigation ──────────────────────────────────────────
function go(page) {
  view.value = page
  profileUser.value = null
  authError.value = ''
  window.location.hash = page
  window.scrollTo(0, 0)
}

// ─── Tasks ───────────────────────────────────────────────
async function openTask(task) {
  selectedTask.value = task
  view.value = 'task-detail'
  window.location.hash = 'task-detail'
  window.scrollTo(0, 0)
  await loadTaskSubmissions(task.id)
}

async function postTask() {
  const f = taskForm.value
  if (!f.title || !f.description) { postTaskError.value = 'Title and description are required'; return }
  if (!currentUser.value?.id) { postTaskError.value = 'You must be logged in'; return }
  postTaskLoading.value = true
  postTaskError.value = ''
  postTaskSuccess.value = ''
  try {
    const data = await apiFetch('/tasks', {
      method: 'POST',
      body: JSON.stringify({ title: f.title, description: f.description, difficulty: f.difficulty, reward_points: Number(f.points), company_id: currentUser.value.id, category: f.category, tags: f.tags }),
    })
    tasks.value.unshift({
      id: data.id, title: data.title, description: data.description,
      category: data.category || f.category, difficulty: data.difficulty,
      tags: f.tags.split(',').map(t => t.trim()).filter(Boolean),
      company: currentUser.value.username, company_id: currentUser.value.id,
      points: data.reward_points, submissions: 0, upvotes: 0,
    })
    postTaskSuccess.value = '// task published successfully!'
    taskForm.value = { title: '', description: '', category: 'Authentication', difficulty: 'Medium', tags: '', points: 500 }
    setTimeout(() => go('tasks'), 1200)
  } catch (e) {
    postTaskError.value = e.message
  } finally {
    postTaskLoading.value = false
  }
}

// ─── Submissions ─────────────────────────────────────────
async function submitFinding() {
  if (!newSubmission.value.trim() || !currentUser.value) return
  submitLoading.value = true
  try {
    const data = await apiFetch(`/tasks/${selectedTask.value.id}/submissions`, {
      method: 'POST',
      body: JSON.stringify({ task_id: selectedTask.value.id, user_id: currentUser.value.id, content: newSubmission.value, status: 'pending' }),
    })
    taskSubmissions.value.unshift({ id: data.id, user: currentUser.value.username, time: 'just now', votes: 0, status: null, content: newSubmission.value, code: null })
    selectedTask.value.submissions++
    if (currentUser.value) currentUser.value.findings = (currentUser.value.findings || 0) + 1
    newSubmission.value = ''
  } catch (e) {
    console.error('Submit failed', e)
  } finally {
    submitLoading.value = false
  }
}

async function upvoteSubmission(post) {
  try {
    const data = await apiFetch(`/submissions/${post.id}/upvote`, { method: 'POST' })
    post.votes = data.upvotes
  } catch {
    post.votes++
  }
}

async function rewardSubmission(post) {
  try {
    await apiFetch(`/submissions/${post.id}/reward`, { method: 'POST', body: JSON.stringify({ badge_type: 'Security Finding' }) })
    post.status = 'rewarded'
    await loadLeaderboard()
  } catch (e) {
    alert(e.message)
  }
}

// ─── Profile ─────────────────────────────────────────────
async function viewUserProfile(username) {
  try {
    const data = await apiFetch(`/users/by-username/${username}`)
    profileUser.value = await loadUserProfile(data.id)
  } catch {
    profileUser.value = { username, reputation: 0, findings: 0, badges: [], role: 'user' }
  }
  go('profile')
}

// ─── Badge award ─────────────────────────────────────────
async function awardBadge() {
  if (!badgeForm.value.user || !badgeForm.value.badge) {
    badgeSuccess.value = 'Please select both a researcher and a badge'
    return
  }
  try {
    const userData = await apiFetch(`/users/by-username/${badgeForm.value.user}`)
    await apiFetch(`/users/${userData.id}/badges`, {
      method: 'POST',
      body: JSON.stringify({ badge_type: badgeForm.value.badge }),
    })
    badgeSuccess.value = `✓ "${badgeForm.value.badge}" awarded to ${badgeForm.value.user}!`
    badgeForm.value.badge = ''
    await loadLeaderboard()
    setTimeout(() => badgeSuccess.value = '', 3000)
  } catch (e) {
    badgeSuccess.value = `Error: ${e.message}`
  }
}

// ─── Init ────────────────────────────────────────────────
onMounted(() => {
  const hash = window.location.hash.replace('#', '')
  if (hash) view.value = hash

  window.addEventListener('popstate', () => {
    const h = window.location.hash.replace('#', '')
    if (h) view.value = h
  })

  loadTasks()
  loadLeaderboard()
})
</script>

<style>
@import url('https://fonts.googleapis.com/css2?family=Space+Mono:wght@400;700&family=DM+Sans:wght@300;400;500;600&display=swap');

*, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

:root {
  --bg: #f8f7f4; --bg2: #f0efe9; --surface: #ffffff;
  --border: #e2e0d8; --border2: #d0cec4;
  --text: #1a1916; --text2: #5a584f; --text3: #8a887e;
  --accent: #1a6b3c; --accent2: #22c55e;
  --accent-bg: #f0fdf4; --accent-border: #bbf7d0;
  --red: #dc2626;
  --mono: 'Space Mono', monospace; --sans: 'DM Sans', sans-serif;
  --radius: 8px;
  --shadow: 0 1px 3px rgba(0,0,0,0.08), 0 1px 2px rgba(0,0,0,0.04);
  --shadow-md: 0 4px 12px rgba(0,0,0,0.08);
}

html { font-size: 15px; }
body { background: var(--bg); color: var(--text); font-family: var(--sans); line-height: 1.6; }
.loading-text { font-family: var(--mono); font-size: 0.75rem; color: var(--text3); padding: 2rem 0; }
.error-text { font-family: var(--mono); font-size: 0.75rem; color: var(--red); padding: 2rem 0; }
.scanlines { position: fixed; inset: 0; pointer-events: none; z-index: 9999; background: repeating-linear-gradient(0deg, transparent, transparent 2px, rgba(0,0,0,0.015) 2px, rgba(0,0,0,0.015) 4px); }

.nav { position: sticky; top: 0; z-index: 100; background: rgba(248,247,244,0.92); backdrop-filter: blur(12px); border-bottom: 1px solid var(--border); padding: 0 1.5rem; }
.nav-inner { max-width: 1200px; margin: 0 auto; display: flex; align-items: center; justify-content: space-between; height: 56px; gap: 2rem; }
.logo { font-family: var(--mono); font-size: 1rem; font-weight: 700; color: var(--text); background: none; border: none; cursor: pointer; }
.logo-bracket { color: var(--text3); } .logo-accent { color: var(--accent); } .logo-small { font-family: var(--mono); font-size: 0.75rem; color: var(--text3); }
.nav-links { display: flex; gap: 0.25rem; align-items: center; }
.nav-links button { background: none; border: none; padding: 0.4rem 0.75rem; font-family: var(--mono); font-size: 0.75rem; color: var(--text2); cursor: pointer; border-radius: var(--radius); transition: all 0.15s; }
.nav-links button:hover, .nav-links button.active { color: var(--accent); background: var(--accent-bg); }
.nav-actions { display: flex; gap: 0.5rem; align-items: center; }

.btn-primary { background: var(--accent); color: #fff; border: none; padding: 0.5rem 1.1rem; border-radius: var(--radius); font-family: var(--mono); font-size: 0.75rem; font-weight: 700; cursor: pointer; transition: all 0.15s; }
.btn-primary:hover:not(:disabled) { background: #15532e; transform: translateY(-1px); }
.btn-primary:disabled { opacity: 0.6; cursor: not-allowed; transform: none; }
.btn-primary.btn-lg { padding: 0.7rem 1.5rem; font-size: 0.8rem; }
.btn-primary.btn-full { width: 100%; padding: 0.75rem; margin-top: 0.5rem; }
.btn-ghost { background: none; border: 1px solid var(--border); padding: 0.4rem 0.9rem; border-radius: var(--radius); font-family: var(--mono); font-size: 0.75rem; color: var(--text2); cursor: pointer; transition: all 0.15s; }
.btn-ghost:hover { border-color: var(--accent); color: var(--accent); }
.btn-outline { background: none; border: 1.5px solid var(--border2); padding: 0.7rem 1.5rem; border-radius: var(--radius); font-family: var(--mono); font-size: 0.8rem; color: var(--text); cursor: pointer; transition: all 0.15s; }
.btn-outline:hover { border-color: var(--accent); color: var(--accent); }
.user-pill { display: flex; align-items: center; gap: 0.5rem; background: var(--accent-bg); border: 1px solid var(--accent-border); padding: 0.35rem 0.75rem; border-radius: 100px; font-family: var(--mono); font-size: 0.72rem; color: var(--accent); cursor: pointer; }
.user-dot { width: 7px; height: 7px; border-radius: 50%; background: var(--accent2); display: inline-block; }

.main { max-width: 1200px; margin: 0 auto; padding: 2.5rem 1.5rem 4rem; min-height: calc(100vh - 56px - 56px); }
.input { width: 100%; background: var(--surface); border: 1.5px solid var(--border); border-radius: var(--radius); padding: 0.6rem 0.85rem; font-family: var(--sans); font-size: 0.9rem; color: var(--text); transition: border-color 0.15s; outline: none; }
.input:focus { border-color: var(--accent); }
.input--sm { width: auto; flex: 1; }
.textarea { resize: vertical; min-height: 120px; }
select.input { cursor: pointer; }
.section-label { font-family: var(--mono); font-size: 0.7rem; color: var(--text3); letter-spacing: 0.05em; margin-bottom: 0.6rem; }

.tag { font-family: var(--mono); font-size: 0.65rem; background: var(--bg2); border: 1px solid var(--border); color: var(--text2); padding: 0.15rem 0.5rem; border-radius: 4px; }
.difficulty-badge { font-family: var(--mono); font-size: 0.65rem; font-weight: 700; padding: 0.15rem 0.55rem; border-radius: 4px; text-transform: uppercase; letter-spacing: 0.05em; }
.difficulty-badge.easy { background: #dcfce7; color: #15803d; } .difficulty-badge.medium { background: #fef9c3; color: #92400e; }
.difficulty-badge.hard { background: #ffedd5; color: #c2410c; } .difficulty-badge.critical { background: #fee2e2; color: #b91c1c; }
.difficulty-text.easy { color: #15803d; } .difficulty-text.medium { color: #92400e; } .difficulty-text.hard { color: #c2410c; } .difficulty-text.critical { color: #b91c1c; }

.landing { display: flex; flex-direction: column; gap: 5rem; }
.hero { max-width: 780px; }
.hero-tag { font-family: var(--mono); font-size: 0.72rem; color: var(--accent); margin-bottom: 1.2rem; }
.hero-title { font-size: clamp(2.4rem, 5vw, 4rem); font-weight: 600; line-height: 1.1; letter-spacing: -0.03em; margin-bottom: 1.2rem; }
.accent { color: var(--accent); }
.hero-sub { font-size: 1.05rem; color: var(--text2); max-width: 560px; margin-bottom: 2rem; line-height: 1.7; }
.hero-cta { display: flex; flex-direction: row; gap: 0.75rem; margin-bottom: 3rem; flex-wrap: wrap; align-items: center; justify-content: flex-start; }
.hero-stats { display: flex; align-items: center; gap: 2rem; flex-wrap: wrap; }
.stat { display: flex; flex-direction: column; }
.stat-num { font-family: var(--mono); font-size: 1.5rem; font-weight: 700; color: var(--text); }
.stat-label { font-size: 0.75rem; color: var(--text3); }
.stat-div { width: 1px; height: 32px; background: var(--border2); }
.features { display: grid; grid-template-columns: repeat(auto-fit, minmax(280px, 1fr)); gap: 1.25rem; }
.feature-card { background: var(--surface); border: 1.5px solid var(--border); border-radius: 12px; padding: 1.5rem; transition: border-color 0.2s, box-shadow 0.2s; }
.feature-card:hover { border-color: var(--accent); box-shadow: var(--shadow-md); }
.feature-card--accent { border-color: var(--accent-border); background: var(--accent-bg); }
.feature-icon { font-size: 1.5rem; margin-bottom: 0.75rem; }
.feature-card h3 { font-size: 0.95rem; font-weight: 600; margin-bottom: 0.5rem; }
.feature-card p { font-size: 0.85rem; color: var(--text2); line-height: 1.65; }
.feed-list { display: flex; flex-direction: column; gap: 0.5rem; }
.feed-item { font-family: var(--mono); font-size: 0.72rem; display: flex; gap: 0.75rem; align-items: baseline; padding: 0.5rem 0; border-bottom: 1px solid var(--border); flex-wrap: wrap; }
.feed-time { color: var(--text3); min-width: 56px; } .feed-user { color: var(--accent); font-weight: 700; } .feed-action { color: var(--text2); } .feed-target { color: var(--text); }

.auth-page { display: flex; justify-content: center; padding: 2rem 0; }
.auth-card { background: var(--surface); border: 1.5px solid var(--border); border-radius: 14px; padding: 2.5rem; width: 100%; max-width: 440px; display: flex; flex-direction: column; gap: 1.1rem; box-shadow: var(--shadow-md); }
.auth-card--wide { max-width: 620px; }
.auth-card h2 { font-size: 1.5rem; font-weight: 600; letter-spacing: -0.02em; }
.auth-sub { font-size: 0.875rem; color: var(--text2); margin-top: -0.5rem; }
.auth-tag { font-family: var(--mono); font-size: 0.68rem; color: var(--accent); }
.form-group { display: flex; flex-direction: column; gap: 0.4rem; }
.form-group label { font-size: 0.8rem; font-weight: 500; color: var(--text2); }
.form-row { display: grid; grid-template-columns: 1fr 1fr; gap: 1rem; }
.role-picker { display: grid; grid-template-columns: 1fr 1fr; gap: 0.5rem; }
.role-btn { background: var(--bg2); border: 1.5px solid var(--border); border-radius: var(--radius); padding: 0.75rem; font-size: 0.875rem; cursor: pointer; display: flex; align-items: center; justify-content: center; gap: 0.5rem; transition: all 0.15s; }
.role-btn.active { background: var(--accent-bg); border-color: var(--accent); color: var(--accent); font-weight: 600; }
.auth-error { font-size: 0.8rem; color: var(--red); background: #fee2e2; border: 1px solid #fecaca; padding: 0.5rem 0.75rem; border-radius: var(--radius); }
.auth-success { font-size: 0.8rem; color: var(--accent); background: var(--accent-bg); border: 1px solid var(--accent-border); padding: 0.5rem 0.75rem; border-radius: var(--radius); }
.auth-switch { font-size: 0.8rem; color: var(--text2); text-align: center; }
.auth-switch button { background: none; border: none; color: var(--accent); cursor: pointer; font-weight: 600; }
.demo-logins { border-top: 1px solid var(--border); padding-top: 1rem; display: flex; flex-direction: column; gap: 0.5rem; }
.demo-label { font-family: var(--mono); font-size: 0.68rem; color: var(--text3); }
.demo-btn { background: var(--bg2); border: 1px solid var(--border); border-radius: var(--radius); padding: 0.5rem; font-size: 0.8rem; cursor: pointer; color: var(--text2); transition: all 0.15s; }
.demo-btn:hover { border-color: var(--accent); color: var(--accent); }

.page-header { display: flex; align-items: flex-end; justify-content: space-between; margin-bottom: 1.5rem; gap: 1rem; }
.page-header h2 { font-size: 1.5rem; font-weight: 600; letter-spacing: -0.02em; }
.filters { display: flex; gap: 0.4rem; flex-wrap: wrap; margin-bottom: 1.5rem; }
.filter-btn { font-family: var(--mono); font-size: 0.7rem; background: var(--surface); border: 1.5px solid var(--border); border-radius: 100px; padding: 0.3rem 0.8rem; color: var(--text2); cursor: pointer; transition: all 0.15s; }
.filter-btn.active, .filter-btn:hover { background: var(--accent-bg); border-color: var(--accent); color: var(--accent); }
.tasks-grid { display: grid; grid-template-columns: repeat(auto-fill, minmax(320px, 1fr)); gap: 1.1rem; }
.tasks-grid--sm { grid-template-columns: repeat(auto-fill, minmax(260px, 1fr)); }
.task-card { background: var(--surface); border: 1.5px solid var(--border); border-radius: 12px; padding: 1.25rem; cursor: pointer; transition: all 0.15s; display: flex; flex-direction: column; gap: 0.75rem; }
.task-card:hover { border-color: var(--accent); box-shadow: var(--shadow-md); transform: translateY(-2px); }
.task-card-header { display: flex; justify-content: space-between; align-items: center; }
.task-pts { font-family: var(--mono); font-size: 0.72rem; font-weight: 700; color: var(--accent); }
.task-title { font-size: 0.92rem; font-weight: 600; line-height: 1.4; }
.task-desc { font-size: 0.82rem; color: var(--text2); line-height: 1.55; display: -webkit-box; -webkit-line-clamp: 2; -webkit-box-orient: vertical; overflow: hidden; }
.task-meta { display: flex; justify-content: space-between; align-items: center; gap: 0.5rem; flex-wrap: wrap; }
.task-company { font-family: var(--mono); font-size: 0.68rem; color: var(--text3); }
.task-tags { display: flex; gap: 0.3rem; flex-wrap: wrap; }
.task-footer { display: flex; gap: 1rem; align-items: center; }
.task-stat { font-family: var(--mono); font-size: 0.68rem; color: var(--text3); }

.back-btn { background: none; border: none; font-family: var(--mono); font-size: 0.75rem; color: var(--text2); cursor: pointer; margin-bottom: 1.5rem; display: block; }
.back-btn:hover { color: var(--accent); }
.task-detail-header { background: var(--surface); border: 1.5px solid var(--border); border-radius: 12px; padding: 1.75rem; margin-bottom: 1.75rem; }
.task-detail-meta { display: flex; align-items: center; gap: 0.75rem; margin-bottom: 0.75rem; flex-wrap: wrap; }
.task-pts-lg { font-family: var(--mono); font-size: 0.85rem; font-weight: 700; color: var(--accent); }
.task-detail-header h2 { font-size: 1.5rem; font-weight: 600; letter-spacing: -0.02em; margin-bottom: 0.75rem; }
.task-detail-header p { color: var(--text2); line-height: 1.65; margin-bottom: 1rem; }
.task-detail-tags { display: flex; gap: 0.4rem; flex-wrap: wrap; }
.task-layout { display: grid; grid-template-columns: 1fr 300px; gap: 1.75rem; align-items: start; }
@media (max-width: 768px) { .task-layout { grid-template-columns: 1fr; } }
.thread-section { display: flex; flex-direction: column; gap: 1.25rem; }
.submit-box { background: var(--surface); border: 1.5px solid var(--border); border-radius: 12px; padding: 1.25rem; display: flex; flex-direction: column; gap: 0.75rem; }
.submit-header { display: flex; align-items: center; gap: 0.5rem; font-family: var(--mono); font-size: 0.75rem; color: var(--accent); }
.submit-actions { display: flex; justify-content: flex-end; }
.login-prompt { text-align: center; padding: 1.5rem; background: var(--bg2); border-radius: var(--radius); font-size: 0.875rem; }
.login-prompt button { background: none; border: none; color: var(--accent); cursor: pointer; font-weight: 600; font-size: 0.875rem; }
.thread { display: flex; flex-direction: column; gap: 1rem; }
.thread-post { display: flex; gap: 1rem; }
.post-avatar { width: 34px; height: 34px; border-radius: 50%; background: var(--accent); color: #fff; font-family: var(--mono); font-weight: 700; font-size: 0.85rem; display: flex; align-items: center; justify-content: center; flex-shrink: 0; }
.post-body { flex: 1; background: var(--surface); border: 1.5px solid var(--border); border-radius: 10px; padding: 1rem; }
.post-header { display: flex; align-items: center; gap: 0.75rem; margin-bottom: 0.6rem; flex-wrap: wrap; }
.post-user { background: none; border: none; font-family: var(--mono); font-size: 0.75rem; font-weight: 700; color: var(--accent); cursor: pointer; }
.post-time { font-family: var(--mono); font-size: 0.68rem; color: var(--text3); }
.post-status { font-family: var(--mono); font-size: 0.65rem; font-weight: 700; padding: 0.15rem 0.5rem; border-radius: 4px; text-transform: uppercase; }
.post-status.verified { background: #dcfce7; color: #15803d; } .post-status.helpful { background: #fef9c3; color: #92400e; } .post-status.rewarded { background: #dbeafe; color: #1d4ed8; }
.post-content { font-size: 0.875rem; color: var(--text); line-height: 1.65; margin-bottom: 0.75rem; }
.post-code { background: #1a1916; border-radius: 6px; padding: 0.75rem 1rem; margin-bottom: 0.75rem; }
.post-code pre { font-family: var(--mono); font-size: 0.75rem; color: #86efac; white-space: pre-wrap; }
.post-actions { display: flex; gap: 0.75rem; align-items: center; }
.vote-btn, .reply-btn, .verify-btn { background: none; border: 1px solid var(--border); border-radius: 6px; padding: 0.25rem 0.6rem; font-family: var(--mono); font-size: 0.68rem; cursor: pointer; color: var(--text2); transition: all 0.15s; }
.vote-btn:hover { border-color: var(--accent); color: var(--accent); } .verify-btn:hover { border-color: #15803d; color: #15803d; }

.sidebar-card { background: var(--surface); border: 1.5px solid var(--border); border-radius: 12px; padding: 1.25rem; margin-bottom: 1rem; }
.info-row { display: flex; justify-content: space-between; align-items: center; padding: 0.4rem 0; border-bottom: 1px solid var(--border); font-size: 0.82rem; }
.info-row:last-child { border-bottom: none; }
.info-row span:first-child { color: var(--text2); }
.accent-text { color: var(--accent); font-weight: 600; font-family: var(--mono); }
.contributor { display: flex; align-items: center; gap: 0.6rem; padding: 0.4rem 0; font-size: 0.82rem; }
.contrib-avatar { width: 26px; height: 26px; border-radius: 50%; background: var(--accent); color: #fff; font-family: var(--mono); font-size: 0.72rem; font-weight: 700; display: flex; align-items: center; justify-content: center; flex-shrink: 0; }
.contrib-pts { margin-left: auto; font-family: var(--mono); font-size: 0.72rem; color: var(--accent); }

.profile-header { display: flex; gap: 2rem; align-items: flex-start; margin-bottom: 2.5rem; background: var(--surface); border: 1.5px solid var(--border); border-radius: 14px; padding: 2rem; }
.profile-avatar { width: 80px; height: 80px; border-radius: 50%; background: var(--accent); color: #fff; font-family: var(--mono); font-size: 2rem; font-weight: 700; display: flex; align-items: center; justify-content: center; flex-shrink: 0; }
.profile-info h2 { font-size: 1.5rem; font-weight: 600; margin-bottom: 0.25rem; }
.profile-bio { font-size: 0.875rem; color: var(--text2); margin-bottom: 1rem; }
.profile-stats { display: flex; gap: 2rem; flex-wrap: wrap; }
.pstat { display: flex; flex-direction: column; }
.pstat-num { font-family: var(--mono); font-size: 1.25rem; font-weight: 700; color: var(--accent); }
.pstat span:last-child { font-size: 0.72rem; color: var(--text3); }
.profile-body { display: block; }
.profile-layout { display: grid; grid-template-columns: 1fr 280px; gap: 1.75rem; align-items: start; }
@media (max-width: 768px) { .profile-layout { grid-template-columns: 1fr; } }
.submission-list { display: flex; flex-direction: column; gap: 0.75rem; }
.submission-item { background: var(--surface); border: 1.5px solid var(--border); border-radius: 10px; padding: 1rem; }
.sub-task { font-family: var(--mono); font-size: 0.72rem; color: var(--accent); margin-bottom: 0.4rem; }
.sub-content { font-size: 0.85rem; color: var(--text); margin-bottom: 0.6rem; line-height: 1.55; }
.sub-meta { display: flex; gap: 0.75rem; align-items: center; font-family: var(--mono); font-size: 0.68rem; color: var(--text3); }
.badge-grid { display: grid; grid-template-columns: repeat(2, 1fr); gap: 0.5rem; }
.badge-item { display: flex; flex-direction: column; align-items: center; gap: 0.25rem; background: var(--bg2); border: 1px solid var(--border); border-radius: 8px; padding: 0.6rem 0.4rem; cursor: help; transition: border-color 0.15s; }
.badge-item:hover { border-color: var(--accent); }
.badge-icon { font-size: 1.2rem; } .badge-name { font-family: var(--mono); font-size: 0.6rem; color: var(--text2); text-align: center; line-height: 1.3; }
.skill-bar { margin-bottom: 0.75rem; }
.skill-label { display: flex; justify-content: space-between; font-family: var(--mono); font-size: 0.68rem; color: var(--text2); margin-bottom: 0.3rem; }
.skill-track { height: 4px; background: var(--border); border-radius: 2px; }
.skill-fill { height: 4px; background: var(--accent); border-radius: 2px; transition: width 0.8s ease; }

.podium { display: flex; justify-content: center; align-items: flex-end; gap: 1rem; margin-bottom: 2.5rem; padding: 2rem 0; }
.podium-slot { display: flex; flex-direction: column; align-items: center; gap: 0.4rem; background: var(--surface); border: 1.5px solid var(--border); border-radius: 12px; padding: 1.25rem 1.75rem; cursor: pointer; transition: all 0.15s; }
.podium-slot:hover { border-color: var(--accent); transform: translateY(-3px); }
.podium-1 { border-color: #fbbf24; background: #fffbeb; }
.podium-crown { font-size: 1.25rem; }
.podium-avatar { width: 44px; height: 44px; border-radius: 50%; background: var(--accent); color: #fff; font-family: var(--mono); font-weight: 700; font-size: 1rem; display: flex; align-items: center; justify-content: center; }
.podium-avatar--lg { width: 56px; height: 56px; font-size: 1.25rem; }
.podium-rank { font-family: var(--mono); font-size: 0.72rem; color: var(--text3); }
.podium-name { font-family: var(--mono); font-size: 0.8rem; font-weight: 700; color: var(--text); }
.podium-pts { font-family: var(--mono); font-size: 0.72rem; color: var(--accent); }
.lb-table { background: var(--surface); border: 1.5px solid var(--border); border-radius: 12px; overflow: hidden; }
.lb-row { display: grid; grid-template-columns: 60px 1fr 120px 120px; align-items: center; padding: 0.9rem 1.25rem; border-bottom: 1px solid var(--border); gap: 1rem; cursor: pointer; transition: background 0.1s; }
.lb-row:last-child { border-bottom: none; }
.lb-row:not(.lb-header):hover { background: var(--bg2); }
.lb-header { font-family: var(--mono); font-size: 0.68rem; color: var(--text3); cursor: default; background: var(--bg2); }
.lb-rank { font-family: var(--mono); font-size: 0.8rem; font-weight: 700; color: var(--text2); }
.lb-rank.lb-top { color: var(--accent); }
.lb-user { display: flex; align-items: center; gap: 0.65rem; font-family: var(--mono); font-size: 0.8rem; font-weight: 700; }
.lb-avatar { width: 28px; height: 28px; border-radius: 50%; background: var(--accent); color: #fff; font-size: 0.72rem; font-weight: 700; display: flex; align-items: center; justify-content: center; flex-shrink: 0; }
.lb-pts { font-family: var(--mono); font-size: 0.8rem; font-weight: 700; color: var(--accent); }
.lb-badges { font-family: var(--mono); font-size: 0.72rem; color: var(--text2); }

.dashboard h2 { font-size: 1.5rem; font-weight: 600; letter-spacing: -0.02em; margin-bottom: 1.75rem; }
.dash-grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(160px, 1fr)); gap: 1rem; margin-bottom: 2.5rem; }
.dash-stat-card { background: var(--surface); border: 1.5px solid var(--border); border-radius: 12px; padding: 1.25rem; }
.dash-stat-label { font-family: var(--mono); font-size: 0.68rem; color: var(--text3); margin-bottom: 0.4rem; }
.dash-stat-num { font-family: var(--mono); font-size: 1.75rem; font-weight: 700; color: var(--text); }
.dash-section { margin-bottom: 2.5rem; }
.dash-section-header { display: flex; justify-content: space-between; align-items: center; margin-bottom: 0.75rem; }
.badge-award-form { display: flex; gap: 0.75rem; align-items: center; flex-wrap: wrap; }

.footer { border-top: 1px solid var(--border); background: var(--surface); }
.footer-inner { max-width: 1200px; margin: 0 auto; padding: 1rem 1.5rem; display: flex; justify-content: space-between; align-items: center; font-family: var(--mono); font-size: 0.68rem; color: var(--text3); flex-wrap: wrap; gap: 0.5rem; }
</style>