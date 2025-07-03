// Import additions
import { useState, useEffect } from "react";
import { Card, CardContent } from "@/components/ui/card";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Textarea } from "@/components/ui/textarea";
import { Moon, Sun } from "lucide-react";
import { motion, AnimatePresence } from "framer-motion";

function hashPassword(password) {
  return btoa(password); // Simple base64 encoding for simulation only
}

const leaderboardData = [
  { username: "alice", score: 87 },
  { username: "bob", score: 72 }
];

const cases = { /* same cases object */ };

export default function CaseInterviewSimulator() {
  const [caseType, setCaseType] = useState("");
  const [step, setStep] = useState(0);
  const [answer, setAnswer] = useState("");
  const [responses, setResponses] = useState([]);
  const [showFeedback, setShowFeedback] = useState(false);
  const [darkMode, setDarkMode] = useState(false);
  const [user, setUser] = useState(null);
  const [loginError, setLoginError] = useState("");
  const [username, setUsername] = useState("");
  const [password, setPassword] = useState("");
  const [email, setEmail] = useState("");
  const [isSignUp, setIsSignUp] = useState(false);

  const currentCase = cases[caseType];
  const currentPrompt = currentCase ? currentCase[step] : null;

  useEffect(() => {
    const storedUser = localStorage.getItem("cis_user");
    if (storedUser) {
      setUser(JSON.parse(storedUser));
    }
  }, []);

  const validateEmail = (email) => {
    return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
  };

  const validatePassword = (pwd) => {
    return pwd.length >= 6 && /[A-Z]/.test(pwd) && /[0-9]/.test(pwd);
  };

  const handleLogin = () => {
    const storedUsers = JSON.parse(localStorage.getItem("cis_users") || "[]");
    const hashed = hashPassword(password);
    const found = storedUsers.find(u => u.username === username && u.password === hashed);
    if (found) {
      setUser(found);
      localStorage.setItem("cis_user", JSON.stringify(found));
      setLoginError("");
    } else {
      setLoginError("Invalid credentials");
    }
  };

  const handleSignUp = () => {
    const storedUsers = JSON.parse(localStorage.getItem("cis_users") || "[]");
    if (storedUsers.some(u => u.username === username)) {
      setLoginError("Username already exists");
      return;
    }
    if (!validateEmail(email)) {
      setLoginError("Invalid email format");
      return;
    }
    if (!validatePassword(password)) {
      setLoginError("Password must be at least 6 characters, include a number and a capital letter");
      return;
    }
    const newUser = { username, password: hashPassword(password), email };
    const updatedUsers = [...storedUsers, newUser];
    localStorage.setItem("cis_users", JSON.stringify(updatedUsers));
    setUser(newUser);
    localStorage.setItem("cis_user", JSON.stringify(newUser));
    setLoginError("");
  };

  const logout = () => {
    setUser(null);
    localStorage.removeItem("cis_user");
    restart();
  };

  const handleNext = () => {
    setResponses([...responses, { prompt: currentPrompt.prompt, answer }]);
    setShowFeedback(false);
    setAnswer("");
    setStep(prev => prev + 1);
  };

  const restart = () => {
    setStep(0);
    setAnswer("");
    setResponses([]);
    setShowFeedback(false);
    setCaseType("");
  };

  const exportMarkdown = () => {
    const md = responses
      .map(r => `### ${r.prompt}\n\n**Your Answer:**\n${r.answer}`)
      .join("\n\n---\n\n");
    const blob = new Blob([md], { type: "text/markdown" });
    const url = URL.createObjectURL(blob);
    const a = document.createElement("a");
    a.href = url;
    a.download = `${caseType.replace(/\s+/g, '_').toLowerCase()}_responses.md`;
    a.click();
  };

  useEffect(() => {
    document.documentElement.classList.toggle("dark", darkMode);
  }, [darkMode]);

  return (
    <div className="min-h-screen transition-colors duration-500 bg-gradient-to-br from-white via-slate-100 to-slate-200 dark:from-slate-900 dark:via-slate-800 dark:to-slate-900 py-10 px-6">
      <div className="max-w-3xl mx-auto rounded-2xl shadow-xl p-8 space-y-6 bg-white dark:bg-slate-800">
        <div className="flex items-center justify-between">
          <h1 className="text-3xl font-bold text-slate-800 dark:text-white">🧠 Case Interview Simulator</h1>
          <div className="flex gap-2 items-center">
            {user && <span className="text-sm text-slate-600 dark:text-slate-300">Hi, {user.username}</span>}
            <Button variant="ghost" size="icon" onClick={() => setDarkMode(!darkMode)}>
              {darkMode ? <Sun className="text-yellow-300" /> : <Moon className="text-slate-800" />}
            </Button>
            {user && <Button variant="outline" onClick={logout}>Logout</Button>}
          </div>
        </div>

        {!user && (
          <div className="space-y-4">
            <p className="text-lg font-semibold text-slate-700 dark:text-slate-300">{isSignUp ? "Sign Up" : "Login"} to start</p>
            <Input placeholder="Username" value={username} onChange={(e) => setUsername(e.target.value)} />
            {isSignUp && <Input type="email" placeholder="Email" value={email} onChange={(e) => setEmail(e.target.value)} />}
            <Input type="password" placeholder="Password" value={password} onChange={(e) => setPassword(e.target.value)} />
            <Button onClick={isSignUp ? handleSignUp : handleLogin}>{isSignUp ? "Create Account" : "Log In"}</Button>
            <button className="text-sm text-blue-500 hover:underline" onClick={() => setIsSignUp(!isSignUp)}>
              {isSignUp ? "Already have an account? Log in" : "Don't have an account? Sign up"}
            </button>
            {loginError && <p className="text-red-500 text-sm">{loginError}</p>}
          </div>
        )}

        {/* ... rest remains unchanged ... */}
      </div>
    </div>
  );
}
