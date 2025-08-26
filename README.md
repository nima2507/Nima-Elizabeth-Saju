import React, { useMemo, useState, useEffect, useRef } from "react";
import { motion, AnimatePresence } from "framer-motion";
import { Button } from "@/components/ui/button";
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card";
import { Input } from "@/components/ui/input";
import { Textarea } from "@/components/ui/textarea";
import { MessageCircle, Send, X, Sparkles, User, GraduationCap, Briefcase, Hammer, BookOpen, Award, Languages } from "lucide-react";

// ========
// DATA (Descriptions only — NO links)
// ========
const profile = {
  name: "Nima Elizabeth Saju",
  title: "Computer Science Graduate Student",
  summary:
    "M.Sc. Computer Science student at Universität Paderborn with strong academic foundation (B.Tech 93.4%, College Topper). Interested in Machine Learning, Deep Learning, Cloud Computing, and Embedded Systems. Seeking internships and part-time roles to apply skills in real-world projects.",
  contact: {
    email: "nimasaju299@gmail.com",
    phone: "+49 15563614301",
    location: "Paderborn, Germany",
  },
};

const education = [
  {
    school: "Universität Paderborn, Germany",
    credential: "M.Sc. Computer Science",
    dates: "2025 – Present",
  },
  {
    school: "Mar Baselios Institute of Technology and Science (APJAKTU), India",
    credential: "B.Tech Computer Science (93.4%, College Topper) — Minor in VLSI & Embedded Systems",
    dates: "2020 – 2024",
  },
  {
    school: "Ebenezer Higher Secondary School",
    credential: "Senior Secondary (98.6%)",
    dates: "2019 – 2020",
  },
  {
    school: "St. George Public School",
    credential: "Secondary (93.6%)",
    dates: "2017 – 2018",
  },
];

const experience = [
  {
    role: "Assistant Manager Trainee (Part-time)",
    org: "Maharshi Ayurlab Kerala Pvt. Ltd",
    dates: "Jan 2022 – Jul 2024",
    bullets: [
      "Handled administration, data entry, and document management.",
      "Coordinated with teams to streamline internal processes.",
    ],
  },
  { role: "Python Intern (Online)", org: "Spectrum Softtech Solutions", dates: "May 2023", bullets: ["Worked on introductory Python tasks and scripting."] },
  { role: "Python Intern (Online)", org: "Goldmine Global Delivery Services", dates: "Apr 2021", bullets: ["Learned fundamentals of Python and basic automation."] },
];

const projects = [
  {
    name: "Masked Face Recognition for Home Safety",
    year: "2024",
    stack: "Deep CNN, Python",
    description:
      "Designed a deep convolutional neural network model to recognize masked faces for home security and theft prevention.",
  },
  {
    name: "Heart Disease Prediction (Honour Project)",
    year: "2024",
    stack: "Python, Machine Learning",
    description:
      "Implemented a predictive pipeline using Python to analyze health features and estimate risk of heart disease.",
  },
  {
    name: "Mask Detection System",
    year: "2023",
    stack: "Raspberry Pi, Machine Learning, Python",
    description:
      "Built a Raspberry Pi–based real-time mask detection system for home security using classical ML + lightweight models.",
  },
];

const pubsWorkshops = [
  {
    type: "Publication",
    title: "Mask Detection and Classification in Thermal Face Images",
    venue: "National Conference, Carmel College",
    date: "2024",
  },
  { type: "Seminar", title: "Mask Detection and Classification", venue: "MBITS", date: "2023" },
  { type: "Workshop", title: "Cloud Computing", venue: "NIT Kozhikode", date: "2022" },
  { type: "Workshop", title: "Machine Learning using Python", venue: "Kothamangalam", date: "2023" },
];

const awards = [
  "Honours in Machine Learning (2024)",
  "Minor in VLSI & Embedded Systems (2024)",
  "College Topper (2024)",
];

const skills = {
  programming: ["C", "C++", "Java", "Python (basic)", "HTML", "CSS"],
  languages: ["Malayalam (native)", "English (C1)", "German (B1)", "Hindi"],
  leadership: [
    "Chief Executive Member – CSI Student Chapter (2022–2024)",
    "Student Grievance Redressal Committee Volunteer (2022–2024)",
    "Class Representative (2023–2024)",
  ],
};

// ========
// Simple Local Chatbot (rule-based over the data above)
// ========
const buildKnowledge = () => {
  const store = [];
  // Profile
  store.push({ q: "who are you", a: `I am ${profile.name}, ${profile.title}. ${profile.summary}` });
  store.push({ q: "contact", a: `Email: ${profile.contact.email} | Phone: ${profile.contact.phone} | Location: ${profile.contact.location}` });
  // Education
  store.push({ q: "education", a: education.map(e => `${e.credential} — ${e.school} (${e.dates})`).join("; ") });
  // Experience
  store.push({ q: "experience", a: experience.map(x => `${x.role} at ${x.org} (${x.dates})`).join("; ") });
  // Projects
  store.push({ q: "projects", a: projects.map(p => `${p.name} (${p.year}) — ${p.stack}`).join("; ") });
  // Skills
  store.push({ q: "skills", a: `Programming: ${skills.programming.join(", ")}. Languages: ${skills.languages.join(", ")}. Leadership: ${skills.leadership.join(", ")}.` });
  // Awards
  store.push({ q: "awards", a: awards.join("; ") });
  // Publication
  store.push({ q: "publication", a: pubsWorkshops.filter(x=>x.type==="Publication").map(x=>`${x.title} — ${x.venue} (${x.date})`).join("; ") });
  return store;
};

const knowledgeBase = buildKnowledge();

function bestAnswer(userText) {
  const text = (userText || "").toLowerCase();
  let best = { score: 0, a: null };
  const keywords = [
    { keys: ["who", "about", "profile"], target: "who are you" },
    { keys: ["contact", "email", "phone"], target: "contact" },
    { keys: ["education", "study", "degree", "university"], target: "education" },
    { keys: ["experience", "work", "intern"], target: "experience" },
    { keys: ["project", "portfolio"], target: "projects" },
    { keys: ["skill", "language", "programming"], target: "skills" },
    { keys: ["award", "honour", "topper"], target: "awards" },
    { keys: ["publication", "paper", "conference"], target: "publication" },
  ];
  for (const k of keywords) {
    const match = k.keys.some(kw => text.includes(kw));
    if (match) {
      const item = knowledgeBase.find(x => x.q === k.target);
      if (item) return item.a;
    }
  }
  // Fallback: soft search by token overlap
  for (const item of knowledgeBase) {
    const tokens = item.q.split(/\s+/);
    const score = tokens.reduce((acc, t) => (text.includes(t) ? acc + 1 : acc), 0);
    if (score > best.score) best = { score, a: item.a };
  }
  return best.a || "I can answer questions about profile, education, experience, projects, skills, and awards.";
}

function Chatbot() {
  const [open, setOpen] = useState(false);
  const [input, setInput] = useState("");
  const [messages, setMessages] = useState([
    { sender: "bot", text: "Hi! Ask me about Nima’s education, projects, skills, awards, or contact." },
  ]);
  const endRef = useRef(null);

  useEffect(() => {
    endRef.current?.scrollIntoView({ behavior: "smooth" });
  }, [messages, open]);

  const send = () => {
    const text = input.trim();
    if (!text) return;
    const reply = bestAnswer(text);
    setMessages(m => [...m, { sender: "user", text }, { sender: "bot", text: reply }]);
    setInput("");
  };

  return (
    <>
      <AnimatePresence>
        {open && (
          <motion.div
            initial={{ opacity: 0, y: 20 }}
            animate={{ opacity: 1, y: 0 }}
            exit={{ opacity: 0, y: 20 }}
            className="fixed bottom-24 right-6 w-full max-w-sm z-50"
          >
            <Card className="shadow-2xl rounded-2xl">
              <CardHeader className="flex flex-row items-center justify-between py-3">
                <CardTitle className="text-base font-semibold">Ask Nima’s Portfolio</CardTitle>
                <Button variant="ghost" size="icon" onClick={() => setOpen(false)} aria-label="Close chat">
                  <X className="h-5 w-5" />
                </Button>
              </CardHeader>
              <CardContent className="space-y-3">
                <div className="h-64 overflow-y-auto pr-2 space-y-2">
                  {messages.map((m, i) => (
                    <div key={i} className={`flex ${m.sender === "user" ? "justify-end" : "justify-start"}`}>
                      <div
                        className={`${m.sender === "user" ? "bg-gray-900 text-white" : "bg-gray-100"} px-3 py-2 rounded-xl max-w-[85%] text-sm`}
                      >
                        {m.text}
                      </div>
                    </div>
                  ))}
                  <div ref={endRef} />
                </div>
                <div className="flex gap-2">
                  <Input
                    placeholder="Type your question..."
                    value={input}
                    onChange={(e) => setInput(e.target.value)}
                    onKeyDown={(e) => e.key === "Enter" && send()}
                  />
                  <Button onClick={send} aria-label="Send message">
                    <Send className="h-4 w-4" />
                  </Button>
                </div>
              </CardContent>
            </Card>
          </motion.div>
        )}
      </AnimatePresence>

      <motion.button
        onClick={() => setOpen(!open)}
        className="fixed bottom-6 right-6 rounded-full shadow-xl p-4 bg-black text-white flex items-center gap-2 z-50"
        whileTap={{ scale: 0.95 }}
        aria-label="Open chat"
      >
        <MessageCircle className="h-5 w-5" /> Chat
      </motion.button>
    </>
  );
}

function Section({ icon: Icon, title, children }) {
  return (
    <Card className="rounded-2xl">
      <CardHeader className="pb-2">
        <div className="flex items-center gap-2">
          {Icon && <Icon className="h-5 w-5" />}
          <CardTitle className="text-lg font-semibold">{title}</CardTitle>
        </div>
      </CardHeader>
      <CardContent className="text-sm leading-relaxed space-y-2">{children}</CardContent>
    </Card>
  );
}

export default function Portfolio() {
  const year = new Date().getFullYear();
  return (
    <div className="min-h-screen bg-gradient-to-b from-white to-gray-50">
      <div className="max-w-5xl mx-auto px-4 py-10">
        {/* Header */}
        <header className="mb-8">
          <div className="flex items-start justify-between">
            <div>
              <h1 className="text-3xl font-bold tracking-tight flex items-center gap-2">
                <Sparkles className="h-6 w-6" /> {profile.name}
              </h1>
              <p className="text-gray-600 mt-1">{profile.title}</p>
              <p className="mt-3 max-w-3xl">{profile.summary}</p>
            </div>
            <Card className="rounded-2xl w-full max-w-xs">
              <CardHeader className="pb-2">
                <CardTitle className="text-base">Contact</CardTitle>
              </CardHeader>
              <CardContent className="text-sm space-y-1">
                <div><span className="font-medium">Email:</span> {profile.contact.email}</div>
                <div><span className="font-medium">Phone:</span> {profile.contact.phone}</div>
                <div><span className="font-medium">Location:</span> {profile.contact.location}</div>
              </CardContent>
            </Card>
          </div>
        </header>

        {/* Grid Sections */}
        <div className="grid grid-cols-1 md:grid-cols-3 gap-4">
          <div className="md:col-span-2 space-y-4">
            <Section icon={GraduationCap} title="Education">
              <ul className="list-disc pl-5 space-y-1">
                {education.map((e, idx) => (
                  <li key={idx}>
                    <span className="font-medium">{e.credential}</span> — {e.school} ({e.dates})
                  </li>
                ))}
              </ul>
            </Section>

            <Section icon={Briefcase} title="Experience">
              <div className="space-y-3">
                {experience.map((x, idx) => (
                  <div key={idx}>
                    <div className="font-medium">{x.role}</div>
                    <div className="text-gray-600 text-xs mb-1">{x.org} — {x.dates}</div>
                    <ul className="list-disc pl-5">
                      {x.bullets.map((b, i) => (
                        <li key={i}>{b}</li>
                      ))}
                    </ul>
                  </div>
                ))}
              </div>
            </Section>

            <Section icon={Hammer} title="Projects">
              <div className="grid gap-3">
                {projects.map((p, idx) => (
                  <Card key={idx} className="rounded-xl">
                    <CardHeader className="pb-1">
                      <CardTitle className="text-base">{p.name}</CardTitle>
                    </CardHeader>
                    <CardContent className="text-sm">
                      <div className="text-gray-600 text-xs mb-1">{p.year} • {p.stack}</div>
                      <p>{p.description}</p>
                    </CardContent>
                  </Card>
                ))}
              </div>
            </Section>
          </div>

          <div className="space-y-4">
            <Section icon={BookOpen} title="Publications & Workshops">
              <ul className="list-disc pl-5 space-y-1">
                {pubsWorkshops.map((pw, i) => (
                  <li key={i}>
                    <span className="font-medium">{pw.type}:</span> {pw.title} — {pw.venue} ({pw.date})
                  </li>
                ))}
              </ul>
            </Section>

            <Section icon={Award} title="Awards">
              <ul className="list-disc pl-5 space-y-1">
                {awards.map((a, i) => (
                  <li key={i}>{a}</li>
                ))}
              </ul>
            </Section>

            <Section icon={Languages} title="Skills & Languages">
              <div className="space-y-2">
                <div>
                  <div className="font-medium">Programming</div>
                  <div>{skills.programming.join(", ")}</div>
                </div>
                <div>
                  <div className="font-medium">Languages</div>
                  <div>{skills.languages.join(", ")}</div>
                </div>
                <div>
                  <div className="font-medium">Leadership</div>
                  <ul className="list-disc pl-5">
                    {skills.leadership.map((l, i) => (
                      <li key={i}>{l}</li>
                    ))}
                  </ul>
                </div>
              </div>
            </Section>
          </div>
        </div>

        {/* Footer */}
        <footer className="text-center text-xs text-gray-500 mt-8">
          © {year} {profile.name}. All rights reserved.
        </footer>
      </div>

      {/* Floating Chatbot */}
      <Chatbot />
    </div>
  );
}
