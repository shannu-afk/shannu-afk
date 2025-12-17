import { useEffect, useState, useRef } from "react";
import { 
  Github, Mail, Linkedin, ChevronDown, GraduationCap, Briefcase, 
  MapPin, Trophy, Star, GitFork, Code, Activity, Heart, ExternalLink,
  Sparkles, Zap, Rocket, Terminal
} from "lucide-react";

// ============== PARTICLE SYSTEM ==============
const ParticleCanvas = () => {
  const canvasRef = useRef<HTMLCanvasElement>(null);

  useEffect(() => {
    const canvas = canvasRef.current;
    if (!canvas) return;
    const ctx = canvas.getContext("2d");
    if (!ctx) return;

    let animationId: number;
    const particles: Array<{
      x: number; y: number; vx: number; vy: number;
      size: number; opacity: number; hue: number;
    }> = [];

    const resize = () => {
      canvas.width = window.innerWidth;
      canvas.height = document.documentElement.scrollHeight;
    };
    resize();
    window.addEventListener("resize", resize);

    // Create particles
    for (let i = 0; i < 80; i++) {
      particles.push({
        x: Math.random() * canvas.width,
        y: Math.random() * canvas.height,
        vx: (Math.random() - 0.5) * 0.8,
        vy: (Math.random() - 0.5) * 0.8,
        size: Math.random() * 3 + 1,
        opacity: Math.random() * 0.6 + 0.2,
        hue: Math.random() > 0.5 ? 180 : 300, // Cyan or Magenta
      });
    }

    const animate = () => {
      ctx.clearRect(0, 0, canvas.width, canvas.height);

      particles.forEach((p, i) => {
        p.x += p.vx;
        p.y += p.vy;

        if (p.x < 0) p.x = canvas.width;
        if (p.x > canvas.width) p.x = 0;
        if (p.y < 0) p.y = canvas.height;
        if (p.y > canvas.height) p.y = 0;

        // Glowing particle
        const gradient = ctx.createRadialGradient(p.x, p.y, 0, p.x, p.y, p.size * 3);
        gradient.addColorStop(0, `hsla(${p.hue}, 100%, 50%, ${p.opacity})`);
        gradient.addColorStop(1, `hsla(${p.hue}, 100%, 50%, 0)`);
        ctx.beginPath();
        ctx.arc(p.x, p.y, p.size * 3, 0, Math.PI * 2);
        ctx.fillStyle = gradient;
        ctx.fill();

        // Connect nearby particles
        particles.slice(i + 1).forEach((p2) => {
          const dx = p.x - p2.x;
          const dy = p.y - p2.y;
          const dist = Math.sqrt(dx * dx + dy * dy);
          if (dist < 120) {
            ctx.beginPath();
            ctx.moveTo(p.x, p.y);
            ctx.lineTo(p2.x, p2.y);
            ctx.strokeStyle = `hsla(${p.hue}, 100%, 50%, ${0.15 * (1 - dist / 120)})`;
            ctx.lineWidth = 0.5;
            ctx.stroke();
          }
        });
      });

      animationId = requestAnimationFrame(animate);
    };
    animate();

    return () => {
      window.removeEventListener("resize", resize);
      cancelAnimationFrame(animationId);
    };
  }, []);

  return <canvas ref={canvasRef} className="fixed inset-0 pointer-events-none z-0" />;
};

// ============== FLOATING ORBS ==============
const FloatingOrbs = () => (
  <div className="fixed inset-0 pointer-events-none overflow-hidden z-0">
    <div className="absolute top-20 left-[10%] w-96 h-96 bg-primary/20 rounded-full blur-[100px] animate-[float_8s_ease-in-out_infinite]" />
    <div className="absolute top-[40%] right-[5%] w-80 h-80 bg-secondary/20 rounded-full blur-[100px] animate-[float_10s_ease-in-out_infinite_1s]" />
    <div className="absolute bottom-[20%] left-[20%] w-72 h-72 bg-accent/20 rounded-full blur-[100px] animate-[float_12s_ease-in-out_infinite_2s]" />
    <div className="absolute top-[60%] right-[30%] w-64 h-64 bg-primary/15 rounded-full blur-[80px] animate-[float_9s_ease-in-out_infinite_0.5s]" />
  </div>
);

// ============== GRID BACKGROUND ==============
const GridBackground = () => (
  <div className="fixed inset-0 z-0 opacity-30">
    <div className="absolute inset-0 bg-[linear-gradient(rgba(0,247,255,0.03)_1px,transparent_1px),linear-gradient(90deg,rgba(0,247,255,0.03)_1px,transparent_1px)] bg-[size:60px_60px] [mask-image:radial-gradient(ellipse_80%_50%_at_50%_50%,black,transparent)]" />
  </div>
);

// ============== ANIMATED RINGS ==============
const AnimatedRings = () => (
  <div className="absolute inset-0 flex items-center justify-center pointer-events-none overflow-hidden">
    {[800, 600, 400].map((size, i) => (
      <div
        key={size}
        className="absolute border rounded-full opacity-20"
        style={{
          width: size,
          height: size,
          borderColor: i % 2 === 0 ? "hsl(180, 100%, 50%)" : "hsl(300, 100%, 50%)",
          animation: `spin ${20 + i * 5}s linear infinite ${i % 2 === 0 ? "" : "reverse"}`,
        }}
      />
    ))}
  </div>
);

// ============== TYPING ANIMATION ==============
const useTypingEffect = (texts: string[], typingSpeed = 100, deletingSpeed = 50, pauseTime = 2000) => {
  const [displayText, setDisplayText] = useState("");
  const [textIndex, setTextIndex] = useState(0);
  const [isDeleting, setIsDeleting] = useState(false);

  useEffect(() => {
    const text = texts[textIndex];
    const timeout = setTimeout(() => {
      if (!isDeleting) {
        if (displayText.length < text.length) {
          setDisplayText(text.slice(0, displayText.length + 1));
        } else {
          setTimeout(() => setIsDeleting(true), pauseTime);
        }
      } else {
        if (displayText.length > 0) {
          setDisplayText(displayText.slice(0, -1));
        } else {
          setIsDeleting(false);
          setTextIndex((prev) => (prev + 1) % texts.length);
        }
      }
    }, isDeleting ? deletingSpeed : typingSpeed);
    return () => clearTimeout(timeout);
  }, [displayText, isDeleting, textIndex, texts, typingSpeed, deletingSpeed, pauseTime]);

  return displayText;
};

// ============== SCROLL ANIMATION HOOK ==============
const useScrollAnimation = () => {
  const [scrollY, setScrollY] = useState(0);
  useEffect(() => {
    const handleScroll = () => setScrollY(window.scrollY);
    window.addEventListener("scroll", handleScroll, { passive: true });
    return () => window.removeEventListener("scroll", handleScroll);
  }, []);
  return scrollY;
};

// ============== MAIN COMPONENT ==============
const Index = () => {
  const scrollY = useScrollAnimation();
  const typedText = useTypingEffect([
    "AI & ML Enthusiast 🤖",
    "Full Stack Developer 💻",
    "OpenCV | React | Django",
    "Building the Future with Code! 🚀",
  ]);

  const [mousePos, setMousePos] = useState({ x: 0, y: 0 });
  const [hoveredTech, setHoveredTech] = useState<number | null>(null);

  useEffect(() => {
    const handleMouse = (e: MouseEvent) => {
      setMousePos({ x: e.clientX, y: e.clientY });
    };
    window.addEventListener("mousemove", handleMouse);
    return () => window.removeEventListener("mousemove", handleMouse);
  }, []);

  const technologies = [
    { name: "Python", icon: "python", color: "#3776AB" },
    { name: "Flask", icon: "flask", color: "#000000" },
    { name: "Django", icon: "django", color: "#092E20" },
    { name: "FastAPI", icon: "fastapi", color: "#009688" },
    { name: "React", icon: "react", color: "#61DAFB" },
    { name: "JavaScript", icon: "js", color: "#F7DF1E" },
    { name: "TypeScript", icon: "ts", color: "#3178C6" },
    { name: "HTML5", icon: "html", color: "#E34F26" },
    { name: "CSS3", icon: "css", color: "#1572B6" },
    { name: "Node.js", icon: "nodejs", color: "#339933" },
    { name: "Git", icon: "git", color: "#F05032" },
    { name: "GitHub", icon: "github", color: "#181717" },
    { name: "VS Code", icon: "vscode", color: "#007ACC" },
    { name: "MySQL", icon: "mysql", color: "#4479A1" },
    { name: "MongoDB", icon: "mongodb", color: "#47A248" },
    { name: "OpenCV", icon: "opencv", color: "#5C3EE8" },
  ];

  const aboutItems = [
    { icon: GraduationCap, title: "Education", desc: "B.Tech in CSE (AI & ML)", detail: "NRI Institute of Technology" },
    { icon: Briefcase, title: "Experience", desc: "Software Dev Intern", detail: "BlackBucks | Lernx" },
    { icon: Mail, title: "Contact", desc: "kodalishanmukh03@gmail.com", detail: "Open for opportunities" },
    { icon: MapPin, title: "Focus", desc: "AI, ML, DSA, OpenCV", detail: "Full Stack Development" },
  ];

  const stats = [
    { label: "Repositories", value: "25+", icon: Code },
    { label: "Stars", value: "50+", icon: Star },
    { label: "Forks", value: "15+", icon: GitFork },
    { label: "Contributions", value: "500+", icon: Activity },
  ];

  const socials = [
    { name: "Email", href: "mailto:kodalishanmukh03@gmail.com", icon: Mail, gradient: "from-red-500 to-orange-500" },
    { name: "GitHub", href: "https://github.com/shannu-afk", icon: Github, gradient: "from-gray-500 to-gray-700" },
    { name: "LinkedIn", href: "https://www.linkedin.com/in/shanmukh-chowdary-kodali-b4462526a/", icon: Linkedin, gradient: "from-blue-500 to-blue-700" },
  ];

  return (
    <div className="min-h-screen bg-background text-foreground overflow-x-hidden">
      {/* Background Effects */}
      <ParticleCanvas />
      <FloatingOrbs />
      <GridBackground />

      {/* Mouse follower glow */}
      <div
        className="fixed w-96 h-96 rounded-full pointer-events-none z-10 opacity-20 blur-[100px] transition-all duration-300"
        style={{
          left: mousePos.x - 192,
          top: mousePos.y - 192,
          background: "radial-gradient(circle, hsl(180, 100%, 50%), transparent)",
        }}
      />

      {/* ============== HERO SECTION ============== */}
      <section className="min-h-screen flex flex-col items-center justify-center relative px-4">
        <AnimatedRings />

        <div className="relative z-20 text-center space-y-8">
          {/* Animated Avatar */}
          <div className="relative inline-block group">
            <div className="absolute inset-0 bg-gradient-to-r from-primary via-secondary to-primary rounded-full blur-xl opacity-60 animate-pulse group-hover:opacity-100 transition-opacity" />
            <div className="relative w-36 h-36 rounded-full bg-gradient-to-br from-primary to-secondary p-1 animate-[glow-pulse_3s_ease-in-out_infinite]">
              <div className="w-full h-full rounded-full bg-background flex items-center justify-center overflow-hidden">
                <span className="text-5xl font-bold gradient-text">KS</span>
              </div>
            </div>
            <div className="absolute -bottom-2 -right-2 w-10 h-10 bg-green-500 rounded-full border-4 border-background flex items-center justify-center animate-bounce">
              <Sparkles className="w-5 h-5 text-white" />
            </div>
          </div>

          {/* Code tag */}
          <div className="flex items-center justify-center gap-2 text-primary font-mono text-sm">
            <Terminal className="w-4 h-4 animate-pulse" />
            <span className="animate-pulse">{"<hello world />"}</span>
          </div>

          {/* Name with glow */}
          <h1 className="text-5xl md:text-7xl lg:text-8xl font-bold">
            Hi 👋 I'm{" "}
            <span className="relative">
              <span className="gradient-text text-glow">K. Shanmukh!</span>
              <span className="absolute -inset-2 bg-gradient-to-r from-primary/20 to-secondary/20 blur-2xl -z-10 animate-pulse" />
            </span>
          </h1>

          {/* Typing effect */}
          <div className="h-14 flex items-center justify-center">
            <span className="text-2xl md:text-4xl font-mono text-muted-foreground">
              {typedText}
              <span className="inline-block w-1 h-8 bg-primary ml-1 animate-[pulse_0.8s_ease-in-out_infinite]" />
            </span>
          </div>

          {/* Animated social buttons */}
          <div className="flex gap-4 justify-center pt-4">
            {socials.map((social, i) => (
              <a
                key={social.name}
                href={social.href}
                target="_blank"
                rel="noopener noreferrer"
                className="group relative p-5 rounded-2xl bg-card/30 backdrop-blur-xl border border-border/50 hover:border-primary/50 transition-all duration-500 hover:scale-110 hover:-translate-y-2"
                style={{ animationDelay: `${i * 0.1}s` }}
              >
                <div className="absolute inset-0 bg-gradient-to-br from-primary/10 to-secondary/10 rounded-2xl opacity-0 group-hover:opacity-100 transition-opacity" />
                <social.icon className="w-6 h-6 text-muted-foreground group-hover:text-primary transition-colors relative z-10" />
                <div className="absolute inset-0 rounded-2xl shadow-[0_0_30px_hsl(180,100%,50%,0.3)] opacity-0 group-hover:opacity-100 transition-opacity" />
              </a>
            ))}
          </div>
        </div>

        {/* Scroll indicator */}
        <div className="absolute bottom-10 left-1/2 -translate-x-1/2 flex flex-col items-center gap-2">
          <span className="text-xs text-muted-foreground font-mono">scroll down</span>
          <ChevronDown className="w-6 h-6 text-primary animate-bounce" />
        </div>
      </section>

      {/* ============== ABOUT SECTION ============== */}
      <section className="py-32 px-4 relative">
        <div className="max-w-6xl mx-auto">
          <div className="text-center mb-20 space-y-4">
            <div className="flex items-center justify-center gap-2 text-primary font-mono text-sm">
              <Zap className="w-4 h-4" />
              <span>{"// about me"}</span>
            </div>
            <h2 className="text-5xl md:text-6xl font-bold gradient-text">Who I Am</h2>
            <div className="w-24 h-1 bg-gradient-to-r from-primary to-secondary mx-auto rounded-full" />
          </div>

          <div className="grid md:grid-cols-2 lg:grid-cols-4 gap-6">
            {aboutItems.map((item, i) => (
              <div
                key={item.title}
                className="group relative p-8 rounded-3xl bg-card/30 backdrop-blur-xl border border-border/50 hover:border-primary/50 transition-all duration-500 hover:-translate-y-3"
                style={{ animationDelay: `${i * 0.1}s` }}
              >
                <div className="absolute inset-0 bg-gradient-to-br from-primary/5 to-secondary/5 rounded-3xl opacity-0 group-hover:opacity-100 transition-opacity" />
                <div className="absolute inset-0 rounded-3xl shadow-[0_0_40px_hsl(180,100%,50%,0.2)] opacity-0 group-hover:opacity-100 transition-opacity" />
                
                <div className="relative z-10">
                  <div className="w-16 h-16 rounded-2xl bg-gradient-to-br from-primary/20 to-secondary/20 flex items-center justify-center mb-6 group-hover:scale-110 group-hover:rotate-3 transition-transform">
                    <item.icon className="w-8 h-8 text-primary" />
                  </div>
                  <h3 className="font-bold text-xl mb-2 group-hover:text-primary transition-colors">{item.title}</h3>
                  <p className="text-foreground/90 text-sm mb-1">{item.desc}</p>
                  <p className="text-muted-foreground text-xs">{item.detail}</p>
                </div>
              </div>
            ))}
          </div>
        </div>
      </section>

      {/* ============== TECH STACK ============== */}
      <section className="py-32 px-4 relative overflow-hidden">
        <div className="max-w-6xl mx-auto relative z-10">
          <div className="text-center mb-20 space-y-4">
            <div className="flex items-center justify-center gap-2 text-primary font-mono text-sm">
              <Rocket className="w-4 h-4" />
              <span>{"// tech stack"}</span>
            </div>
            <h2 className="text-5xl md:text-6xl font-bold gradient-text">Technologies</h2>
            <div className="w-24 h-1 bg-gradient-to-r from-primary to-secondary mx-auto rounded-full" />
          </div>

          <div className="grid grid-cols-4 md:grid-cols-6 lg:grid-cols-8 gap-4">
            {technologies.map((tech, i) => (
              <div
                key={tech.name}
                className="group relative"
                onMouseEnter={() => setHoveredTech(i)}
                onMouseLeave={() => setHoveredTech(null)}
              >
                <div
                  className={`aspect-square rounded-2xl bg-card/30 backdrop-blur-xl border border-border/50 p-4 flex flex-col items-center justify-center gap-2 transition-all duration-300 cursor-pointer ${
                    hoveredTech === i ? "scale-110 border-primary/50 -translate-y-2" : "hover:scale-105"
                  }`}
                >
                  <img
                    src={`https://skillicons.dev/icons?i=${tech.icon}`}
                    alt={tech.name}
                    className="w-10 h-10 md:w-12 md:h-12 transition-transform group-hover:scale-110"
                  />
                  <span className="text-[10px] text-muted-foreground opacity-0 group-hover:opacity-100 transition-opacity text-center">
                    {tech.name}
                  </span>
                </div>

                {hoveredTech === i && (
                  <div
                    className="absolute inset-0 rounded-2xl blur-2xl opacity-40 -z-10 animate-pulse"
                    style={{ backgroundColor: tech.color }}
                  />
                )}
              </div>
            ))}
          </div>
        </div>
      </section>

      {/* ============== GITHUB STATS ============== */}
      <section className="py-32 px-4 relative">
        <div className="max-w-6xl mx-auto">
          <div className="text-center mb-20 space-y-4">
            <div className="flex items-center justify-center gap-2 text-primary font-mono text-sm">
              <Activity className="w-4 h-4" />
              <span>{"// github stats"}</span>
            </div>
            <h2 className="text-5xl md:text-6xl font-bold gradient-text">My GitHub Journey</h2>
            <div className="w-24 h-1 bg-gradient-to-r from-primary to-secondary mx-auto rounded-full" />
          </div>

          {/* Quick stats */}
          <div className="grid grid-cols-2 md:grid-cols-4 gap-4 mb-12">
            {stats.map((stat, i) => (
              <div
                key={stat.label}
                className="group relative p-8 rounded-3xl bg-card/30 backdrop-blur-xl border border-border/50 text-center hover:border-primary/50 transition-all duration-500 hover:-translate-y-2"
              >
                <div className="absolute inset-0 rounded-3xl shadow-[0_0_40px_hsl(180,100%,50%,0.2)] opacity-0 group-hover:opacity-100 transition-opacity" />
                <stat.icon className="w-10 h-10 mx-auto mb-4 text-primary group-hover:scale-110 transition-transform" />
                <p className="text-4xl font-bold gradient-text mb-2">{stat.value}</p>
                <p className="text-sm text-muted-foreground">{stat.label}</p>
              </div>
            ))}
          </div>

          {/* GitHub cards */}
          <div className="grid md:grid-cols-2 gap-6 mb-8">
            <div className="rounded-3xl bg-card/30 backdrop-blur-xl border border-border/50 p-6 hover:border-primary/50 transition-all duration-500 overflow-hidden group">
              <div className="absolute inset-0 shadow-[0_0_60px_hsl(180,100%,50%,0.1)] opacity-0 group-hover:opacity-100 transition-opacity" />
              <img
                src="https://github-readme-stats.vercel.app/api?username=shannu-afk&show_icons=true&theme=transparent&hide_border=true&title_color=00f7ff&icon_color=ff00ff&text_color=ffffff&bg_color=00000000"
                alt="GitHub Stats"
                className="w-full h-auto"
              />
            </div>
            <div className="rounded-3xl bg-card/30 backdrop-blur-xl border border-border/50 p-6 hover:border-primary/50 transition-all duration-500 overflow-hidden group">
              <img
                src="https://github-readme-stats.vercel.app/api/top-langs/?username=shannu-afk&layout=compact&theme=transparent&hide_border=true&title_color=00f7ff&text_color=ffffff&bg_color=00000000"
                alt="Top Languages"
                className="w-full h-auto"
              />
            </div>
          </div>

          {/* Streak */}
          <div className="rounded-3xl bg-card/30 backdrop-blur-xl border border-border/50 p-6 hover:border-primary/50 transition-all duration-500 overflow-hidden">
            <img
              src="https://github-readme-streak-stats.herokuapp.com/?user=shannu-afk&theme=transparent&hide_border=true&stroke=00f7ff&ring=ff00ff&fire=ff00ff&currStreakLabel=00f7ff&sideLabels=00f7ff&currStreakNum=ffffff&sideNums=ffffff&dates=888888&background=00000000"
              alt="GitHub Streak"
              className="w-full h-auto max-w-3xl mx-auto"
            />
          </div>
        </div>
      </section>

      {/* ============== ACTIVITY GRAPH ============== */}
      <section className="py-32 px-4 relative">
        <div className="max-w-6xl mx-auto">
          <div className="text-center mb-20 space-y-4">
            <span className="text-primary font-mono text-sm">{"// contributions"}</span>
            <h2 className="text-5xl md:text-6xl font-bold gradient-text">Activity Graph</h2>
            <div className="w-24 h-1 bg-gradient-to-r from-primary to-secondary mx-auto rounded-full" />
          </div>

          <div className="rounded-3xl bg-card/30 backdrop-blur-xl border border-border/50 p-6 hover:border-primary/50 transition-all duration-500 overflow-hidden">
            <img
              src="https://github-readme-activity-graph.vercel.app/graph?username=shannu-afk&theme=react-dark&hide_border=true&area=true&bg_color=00000000&color=00f7ff&line=ff00ff&point=ffffff&area_color=00f7ff"
              alt="Activity Graph"
              className="w-full h-auto"
            />
          </div>
        </div>
      </section>

      {/* ============== TROPHIES ============== */}
      <section className="py-32 px-4 relative">
        <div className="max-w-6xl mx-auto">
          <div className="text-center mb-20 space-y-4">
            <div className="flex items-center justify-center gap-2 text-primary font-mono text-sm">
              <Trophy className="w-4 h-4" />
              <span>{"// achievements"}</span>
            </div>
            <h2 className="text-5xl md:text-6xl font-bold gradient-text">GitHub Trophies</h2>
            <div className="w-24 h-1 bg-gradient-to-r from-primary to-secondary mx-auto rounded-full" />
          </div>

          <div className="rounded-3xl bg-card/30 backdrop-blur-xl border border-border/50 p-6 hover:border-primary/50 transition-all duration-500 overflow-hidden">
            <img
              src="https://github-profile-trophy.vercel.app/?username=shannu-afk&theme=discord&no-frame=true&no-bg=true&column=-1&margin-w=15"
              alt="GitHub Trophies"
              className="w-full h-auto"
            />
          </div>
        </div>
      </section>

      {/* ============== CONTACT ============== */}
      <section className="py-32 px-4 relative overflow-hidden">
        <div className="max-w-4xl mx-auto relative z-10">
          <div className="text-center mb-20 space-y-4">
            <span className="text-primary font-mono text-sm">{"// let's connect"}</span>
            <h2 className="text-5xl md:text-6xl font-bold gradient-text">Get In Touch</h2>
            <div className="w-24 h-1 bg-gradient-to-r from-primary to-secondary mx-auto rounded-full" />
            <p className="text-muted-foreground max-w-xl mx-auto pt-4">
              Always open for new opportunities, collaborations, or just a chat about tech!
            </p>
          </div>

          <div className="grid md:grid-cols-3 gap-6">
            {socials.map((social) => (
              <a
                key={social.name}
                href={social.href}
                target="_blank"
                rel="noopener noreferrer"
                className="group relative p-10 rounded-3xl bg-card/30 backdrop-blur-xl border border-border/50 hover:border-primary/50 transition-all duration-500 hover:-translate-y-3 text-center"
              >
                <div className="absolute inset-0 rounded-3xl shadow-[0_0_60px_hsl(180,100%,50%,0.2)] opacity-0 group-hover:opacity-100 transition-opacity" />
                
                <div className={`w-20 h-20 mx-auto rounded-3xl bg-gradient-to-br ${social.gradient} flex items-center justify-center mb-6 group-hover:scale-110 group-hover:rotate-6 transition-transform shadow-2xl`}>
                  <social.icon className="w-10 h-10 text-white" />
                </div>
                
                <h3 className="font-bold text-2xl mb-2 group-hover:text-primary transition-colors flex items-center justify-center gap-2">
                  {social.name}
                  <ExternalLink className="w-5 h-5 opacity-0 group-hover:opacity-100 transition-opacity" />
                </h3>
              </a>
            ))}
          </div>
        </div>
      </section>

      {/* ============== FOOTER ============== */}
      <footer className="py-16 px-4 relative overflow-hidden">
        <div className="absolute top-0 left-0 right-0 h-px bg-gradient-to-r from-transparent via-primary/50 to-transparent" />

        <div className="max-w-6xl mx-auto text-center space-y-8">
          {/* Animated bars */}
          <div className="flex justify-center gap-1">
            {[...Array(7)].map((_, i) => (
              <div
                key={i}
                className="w-1 bg-gradient-to-t from-primary to-secondary rounded-full"
                style={{
                  height: `${20 + Math.sin(i) * 15}px`,
                  animation: `pulse 1s ease-in-out infinite`,
                  animationDelay: `${i * 0.1}s`,
                }}
              />
            ))}
          </div>

          <p className="text-muted-foreground flex items-center justify-center gap-3 flex-wrap text-lg">
            <span>Crafted with</span>
            <Heart className="w-5 h-5 text-secondary animate-pulse" />
            <span>and</span>
            <Code className="w-5 h-5 text-primary" />
            <span>by</span>
            <span className="gradient-text font-bold text-xl">K. Shanmukh</span>
          </p>

          <p className="text-muted-foreground/50 text-sm">
            © {new Date().getFullYear()} All rights reserved.
          </p>
        </div>
      </footer>
    </div>
  );
};

export default Index;
