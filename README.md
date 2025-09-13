import React, { useState, useRef } from "react";
import { motion, AnimatePresence } from "framer-motion";

// اللهجة التونسية Strings
const STRINGS = {
  title: "منظم وقتي في القراية",
  addSubject: "زيد مادة",
  subjectPlaceholder: "إسم المادة",
  addLesson: "زيد درس",
  lessonPlaceholder: "إسم الدرس",
  weeklyObjective: "هدف الأسبوع",
  hours: "ساعات",
  setObjective: "حدد ساعات المراجعة",
  startTimer: "إبدا المراجعة",
  stopTimer: "وقف",
  points: "النقاط",
  studied: "ساعات قريتها",
  inviteFriend: "عزم صاحبك",
  friendPlaceholder: "إسم صاحبك",
  challenge: "تحدي: شكون يراجع أكثر ساعات الأسبوع هذا؟",
  leaderboard: "الترتيب",
  you: "إنت",
  lessons: "الدروس",
  subjects: "المواد",
  studyNow: "راجع توة"
};

const colors = [
  "bg-yellow-100",
  "bg-rose-100",
  "bg-blue-100",
  "bg-green-100",
  "bg-pink-100",
  "bg-purple-100"
];

export default function App() {
  // Section: Subjects & Lessons
  const [subjects, setSubjects] = useState([]);
  const [subjectInput, setSubjectInput] = useState("");
  const [lessonInputs, setLessonInputs] = useState({});

  // Section: Weekly Objective
  const [objective, setObjective] = useState(4);
  const [studiedHours, setStudiedHours] = useState(0);

  // Section: Points & Timer
  const [timerOn, setTimerOn] = useState(false);
  const [timerSubject, setTimerSubject] = useState(null);
  const timerRef = useRef(null);
  const [subjectHours, setSubjectHours] = useState({});
  const [points, setPoints] = useState(0);

  // Section: Friends & Challenges
  const [friends, setFriends] = useState([{ name: STRINGS.you, hours: 0, points: 0, isYou: true }]);
  const [friendInput, setFriendInput] = useState("");
  const [challengeActive, setChallengeActive] = useState(false);

  // Animations
  const fadeIn = { initial: { opacity: 0, y: 20 }, animate: { opacity: 1, y: 0 } };

  // Helper: Add Subject
  function handleAddSubject() {
    if (!subjectInput.trim()) return;
    setSubjects([...subjects, { name: subjectInput, lessons: [] }]);
    setSubjectInput("");
  }

  // Helper: Add Lesson
  function handleAddLesson(idx) {
    if (!lessonInputs[idx] || !lessonInputs[idx].trim()) return;
    const updatedSubjects = subjects.map((subj, i) =>
      i === idx ? { ...subj, lessons: [...subj.lessons, lessonInputs[idx]] } : subj
    );
    setSubjects(updatedSubjects);
    setLessonInputs({ ...lessonInputs, [idx]: "" });
  }

  // Helper: Start/Stop Timer
  function handleStartTimer(subjectIdx) {
    setTimerOn(true);
    setTimerSubject(subjectIdx);
    timerRef.current = setInterval(() => {
      setStudiedHours(h => h + 1 / 60); // +1min
      setSubjectHours(sh => ({
        ...sh,
        [subjectIdx]: (sh[subjectIdx] || 0) + 1 / 60
      }));
      setPoints(p => p + 1); // 1pt/min
      // Update in leaderboard (simulate for you)
      setFriends(fs =>
        fs.map(f =>
          f.isYou
            ? { ...f, hours: studiedHours + 1 / 60, points: points + 1 }
            : f
        )
      );
    }, 1000 * 60); // every minute
  }

  function handleStopTimer() {
    setTimerOn(false);
    setTimerSubject(null);
    clearInterval(timerRef.current);
  }

  // Helper: Invite Friend
  function handleInviteFriend() {
    if (!friendInput.trim()) return;
    setFriends([
      ...friends,
      {
        name: friendInput,
        hours: Math.floor(Math.random() * 5 + 1), // random hours
        points: Math.floor(Math.random() * 100 + 10)
      }
    ]);
    setFriendInput("");
    setChallengeActive(true);
  }

  // UI
  return (
    <div className="min-h-screen bg-gradient-to-br from-yellow-100 via-rose-50 to-blue-100 p-0 m-0 font-sans">
      {/* Header */}
      <motion.header {...fadeIn} className="py-6 px-4 flex justify-between items-center">
        <h1 className="text-3xl md:text-5xl font-bold text-rose-600 animate-bounce">
          {STRINGS.title} 🕒
        </h1>
        <div>
          <span className="rounded-full bg-blue-200 px-3 py-2 font-semibold text-blue-700 shadow">
            {STRINGS.points}: {points}
          </span>
        </div>
      </motion.header>
      <main className="max-w-4xl mx-auto px-4">
        {/* Weekly Objective */}
        <motion.section {...fadeIn} transition={{ delay: 0.1 }} className="mb-4">
          <div className="bg-white rounded-xl shadow-lg p-4 flex flex-col md:flex-row gap-4 items-center">
            <h2 className="font-bold text-xl text-rose-700 flex-shrink-0">
              {STRINGS.weeklyObjective}:&nbsp;
              <input
                type="number"
                min={1}
                max={40}
                value={objective}
                onChange={e => setObjective(Number(e.target.value))}
                className="border rounded px-2 py-1 w-16"
              />
              &nbsp;{STRINGS.hours}
            </h2>
            <div className="text-gray-700">
              {STRINGS.studied}:&nbsp;
              <span className="font-bold">{studiedHours.toFixed(2)} {STRINGS.hours}</span>
              {" "}
              ({((studiedHours / objective) * 100).toFixed(1)}%)
            </div>
          </div>
        </motion.section>
        {/* Subjects & Lessons */}
        <motion.section {...fadeIn} transition={{ delay: 0.2 }} className="mb-8">
          <div className="bg-white rounded-xl shadow-lg p-4">
            <h2 className="font-bold text-lg mb-2 text-blue-700">{STRINGS.subjects}</h2>
            <div className="flex gap-2 mb-4">
              <input
                value={subjectInput}
                onChange={e => setSubjectInput(e.target.value)}
                placeholder={STRINGS.subjectPlaceholder}
                className="border rounded px-2 py-1 flex-1"
              />
              <button
                onClick={handleAddSubject}
                className="bg-rose-400 text-white rounded px-3 py-1 font-bold hover:bg-rose-500 transition"
              >
                {STRINGS.addSubject}
              </button>
            </div>
            {/* List Subjects */}
            <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
              {subjects.map((subj, idx) => (
                <motion.div
                  key={idx}
                  layout
                  initial={{ x: -20, opacity: 0 }}
                  animate={{ x: 0, opacity: 1 }}
                  className={`rounded-lg p-3 shadow ${colors[idx % colors.length]}`}
                >
                  <div className="flex justify-between items-center mb-2">
                    <span className="font-semibold text-xl text-rose-700">{subj.name}</span>
                    <span className="bg-blue-200 px-2 py-1 rounded text-blue-700 text-xs">
                      {STRINGS.studied}: {(subjectHours[idx] || 0).toFixed(2)} {STRINGS.hours}
                    </span>
                  </div>
                  {/* Lessons */}
                  <div className="mb-2">
                    <span className="font-semibold">{STRINGS.lessons}:</span>
                    <ul className="pl-4 list-disc">
                      {subj.lessons.map((lesson, lidx) => (
                        <motion.li key={lidx} layout className="text-gray-700">{lesson}</motion.li>
                      ))}
                    </ul>
                  </div>
                  {/* Add Lesson */}
                  <div className="flex gap-2 mb-2">
                    <input
                      value={lessonInputs[idx] || ""}
                      onChange={e =>
                        setLessonInputs({ ...lessonInputs, [idx]: e.target.value })
                      }
                      placeholder={STRINGS.lessonPlaceholder}
                      className="border rounded px-2 py-1 flex-1"
                    />
                    <button
                      onClick={() => handleAddLesson(idx)}
                      className="bg-blue-400 text-white rounded px-3 py-1 font-bold hover:bg-blue-500"
                    >
                      {STRINGS.addLesson}
                    </button>
                  </div>
                  {/* Study Timer */}
                  <div className="flex gap-3 items-center">
                    {timerOn && timerSubject === idx ? (
                      <button
                        onClick={handleStopTimer}
                        className="bg-gray-300 text-rose-700 rounded px-3 py-1 font-bold"
                      >
                        {STRINGS.stopTimer}
                      </button>
                    ) : (
                      <button
                        onClick={() => handleStartTimer(idx)}
                        className="bg-green-400 text-white rounded px-3 py-1 font-bold hover:bg-green-500"
                      >
                        {STRINGS.studyNow}
                      </button>
                    )}
                  </div>
                </motion.div>
              ))}
            </div>
          </div>
        </motion.section>
        {/* Challenge & Friends */}
        <motion.section {...fadeIn} transition={{ delay: 0.3 }} className="mb-8">
          <div className="bg-white rounded-xl shadow-lg p-4">
            <h2 className="font-bold text-lg mb-2 text-green-700">{STRINGS.challenge}</h2>
            <div className="flex gap-2 mb-4">
              <input
                value={friendInput}
                onChange={e => setFriendInput(e.target.value)}
                placeholder={STRINGS.friendPlaceholder}
                className="border rounded px-2 py-1 flex-1"
              />
              <button
                onClick={handleInviteFriend}
                className="bg-green-400 text-white rounded px-3 py-1 font-bold hover:bg-green-500"
              >
                {STRINGS.inviteFriend}
              </button>
            </div>
            {/* Leaderboard */}
            <div className="bg-blue-100 rounded p-3">
              <h3 className="font-bold text-blue-700 mb-2">{STRINGS.leaderboard}</h3>
              <ul>
                {friends
                  .sort((a, b) => b.hours - a.hours)
                  .map((f, idx) => (
                    <li key={idx} className={`flex justify-between py-1 ${f.isYou ? "font-bold text-rose-700" : ""}`}>
                      <span>{f.name}</span>
                      <span>
                        {STRINGS.hours}: {f.hours.toFixed(2)} | {STRINGS.points}: {f.points}
                      </span>
                    </li>
                  ))}
              </ul>
            </div>
          </div>
        </motion.section>
      </main>
      {/* Footer */}
      <motion.footer {...fadeIn} className="py-6 text-center text-gray-500">
        <span>
          صنعناه بشغف 🇹🇳 | Powered by Copilot | نسخة تجريبية
        </span>
      </motion.footer>
      {/* Tailwind CDN for quick styling */}
      <link
        href="https://cdn.jsdelivr.net/npm/tailwindcss@2.2.19/dist/tailwind.min.css"
        rel="stylesheet"
      />
      {/* Framer Motion CDN for demo if needed */}
      <script src="https://unpkg.com/framer-motion/dist/framer-motion.umd.js"></script>
    </div>
  );
}
