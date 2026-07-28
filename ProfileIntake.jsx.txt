import { useState, useMemo } from "react";
import { Radar, Plane, GraduationCap, Briefcase, Target, Sliders, Link2, ChevronDown, Check, Send, Copy, GripVertical, X } from "lucide-react";

const OPPORTUNITY_TYPES = [
  "Scholarships", "Study abroad", "Fellowships", "Sponsored conferences",
  "Sponsored seminars", "Research grants", "Remote jobs", "International consultancies",
  "NGO opportunities", "UN / WHO / UNICEF roles", "AI-related roles", "Research collaborations",
];

const SKILL_TAGS = [
  "Public Health", "Social & Behaviour Change", "Research", "AI", "Automation",
  "Technical writing", "Training", "Digital tools", "Data analysis", "Communication",
];

function Field({ label, hint, children }) {
  return (
    <label className="block">
      <span className="font-mono text-[11px] tracking-wider uppercase text-slate-400">{label}</span>
      {hint && <span className="block font-mono text-[10px] text-slate-500 mt-0.5">{hint}</span>}
      <div className="mt-2">{children}</div>
    </label>
  );
}

const inputCls =
  "w-full bg-[#1C222C] border border-[#2A3140] rounded-md px-3 py-2.5 text-[15px] text-[#EDEEF0] placeholder:text-slate-500 focus:outline-none focus:ring-2 focus:ring-[#2DD4BF]/50 focus:border-[#2DD4BF]/50 transition-colors";

function Section({ n, title, icon: Icon, children }) {
  return (
    <section className="relative pl-[52px] py-8 border-b border-[#242B37] last:border-0">
      <div className="absolute left-0 top-8 flex flex-col items-center">
        <div className="w-9 h-9 rounded-full bg-[#1C222C] border border-[#2A3140] flex items-center justify-center">
          <Icon size={15} className="text-[#2DD4BF]" strokeWidth={2} />
        </div>
      </div>
      <div className="font-mono text-[11px] tracking-widest text-slate-500 mb-1">SECTION {n}</div>
      <h2 className="text-[19px] font-semibold text-[#EDEEF0] mb-5">{title}</h2>
      <div className="space-y-5">{children}</div>
    </section>
  );
}

function TagToggle({ label, active, onClick }) {
  return (
    <button
      type="button"
      onClick={onClick}
      className={`px-3 py-1.5 rounded-full text-[13px] font-medium border transition-colors ${
        active
          ? "bg-[#2DD4BF]/15 border-[#2DD4BF]/60 text-[#2DD4BF]"
          : "bg-transparent border-[#2A3140] text-slate-400 hover:border-slate-500"
      }`}
    >
      {active && <Check size={12} className="inline mr-1 -mt-0.5" />}
      {label}
    </button>
  );
}

export default function ProfileIntake() {
  const [form, setForm] = useState({
    name: "", email: "", country: "", nationality: "", location: "",
    degrees: "", institutions: "", certifications: "", ongoingLearning: "",
    experience: "",
    skills: [], goals: [],
    remotePref: "Remote preferred", visaNeeded: "Not sure", funded: "Fully funded preferred",
    preferredCountries: "", avoidCountries: "", minFunding: "", otherPrefs: "",
    cvLink: "", linkedin: "", portfolio: "",
    webhookUrl: "",
  });
  const [priorities, setPriorities] = useState([
    "Fully funded scholarships", "International consultancies", "Remote jobs",
    "Fellowships", "Sponsored conferences",
  ]);

  const movePriority = (i, dir) => {
    setPriorities((p) => {
      const next = [...p];
      const j = i + dir;
      if (j < 0 || j >= next.length) return next;
      [next[i], next[j]] = [next[j], next[i]];
      return next;
    });
  };
  const [status, setStatus] = useState(null); // null | "sending" | "sent" | "error" | "copied"

  const set = (k, v) => setForm((f) => ({ ...f, [k]: v }));
  const toggleTag = (k, val) =>
    setForm((f) => ({
      ...f,
      [k]: f[k].includes(val) ? f[k].filter((x) => x !== val) : [...f[k], val],
    }));

  const completeness = useMemo(() => {
    const checks = [
      form.name, form.email, form.country, form.location,
      form.degrees, form.institutions,
      form.experience,
      form.skills.length > 0,
      form.goals.length > 0,
      form.cvLink || form.linkedin,
    ];
    const filled = checks.filter(Boolean).length;
    return Math.round((filled / checks.length) * 100);
  }, [form]);

  const bars = 12;
  const litBars = Math.round((completeness / 100) * bars);

  async function handleSave() {
    const payload = { ...form, priorities };
    delete payload.webhookUrl;

    if (!form.webhookUrl) {
      try {
        await navigator.clipboard.writeText(JSON.stringify(payload, null, 2));
        setStatus("copied");
      } catch {
        setStatus("error");
      }
      setTimeout(() => setStatus(null), 2500);
      return;
    }

    setStatus("sending");
    try {
      await fetch(form.webhookUrl, {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify(payload),
        mode: "no-cors",
      });
      setStatus("sent");
    } catch {
      setStatus("error");
    }
    setTimeout(() => setStatus(null), 3000);
  }

  return (
    <div className="min-h-screen bg-[#14181F] text-[#EDEEF0]">
      {/* Header / signal meter */}
      <div className="sticky top-0 z-10 bg-[#14181F]/95 backdrop-blur border-b border-[#242B37]">
        <div className="max-w-2xl mx-auto px-5 py-4">
          <div className="flex items-center justify-between mb-3">
            <div className="flex items-center gap-2">
              <Radar size={18} className="text-[#2DD4BF]" />
              <span className="font-mono text-[12px] tracking-widest text-slate-400">CAREER DNA INTAKE</span>
            </div>
            <span className="font-mono text-[12px] text-[#2DD4BF]">{completeness}%</span>
          </div>
          <div className="flex gap-[3px]">
            {Array.from({ length: bars }).map((_, i) => (
              <div
                key={i}
                className={`h-1.5 flex-1 rounded-full transition-colors duration-300 ${
                  i < litBars ? "bg-[#2DD4BF]" : "bg-[#242B37]"
                }`}
              />
            ))}
          </div>
          <p className="font-mono text-[10px] text-slate-500 mt-2">
            SIGNAL STRENGTH — stronger signal, sharper opportunity matches
          </p>
        </div>
      </div>

      <div className="max-w-2xl mx-auto px-5 pb-16">
        <div className="pt-8 pb-2">
          <h1 className="text-2xl font-bold mb-1">Build your profile</h1>
          <p className="text-slate-400 text-[14px]">
            This feeds the matching engine directly. Fill in what you have — you can always come back and update it.
          </p>
        </div>

        <Section n="01" title="Personal" icon={Plane}>
          <div className="grid grid-cols-2 gap-4">
            <Field label="Full name"><input className={inputCls} value={form.name} onChange={(e) => set("name", e.target.value)} placeholder="Victor Chucks Jim" /></Field>
            <Field label="Email"><input className={inputCls} type="email" value={form.email} onChange={(e) => set("email", e.target.value)} placeholder="you@email.com" /></Field>
            <Field label="Country of residence"><input className={inputCls} value={form.country} onChange={(e) => set("country", e.target.value)} placeholder="Nigeria" /></Field>
            <Field label="Nationality"><input className={inputCls} value={form.nationality} onChange={(e) => set("nationality", e.target.value)} placeholder="Nigerian" /></Field>
          </div>
          <Field label="Current location"><input className={inputCls} value={form.location} onChange={(e) => set("location", e.target.value)} placeholder="City, State" /></Field>
        </Section>

        <Section n="02" title="Education" icon={GraduationCap}>
          <Field label="Degrees" hint="Paste directly — degree, field, year"><textarea className={inputCls + " min-h-[70px]"} value={form.degrees} onChange={(e) => set("degrees", e.target.value)} /></Field>
          <Field label="Institutions"><textarea className={inputCls + " min-h-[60px]"} value={form.institutions} onChange={(e) => set("institutions", e.target.value)} /></Field>
          <Field label="Certifications"><textarea className={inputCls + " min-h-[60px]"} value={form.certifications} onChange={(e) => set("certifications", e.target.value)} /></Field>
          <Field label="Ongoing learning"><textarea className={inputCls + " min-h-[50px]"} value={form.ongoingLearning} onChange={(e) => set("ongoingLearning", e.target.value)} /></Field>
        </Section>

        <Section n="03" title="Experience" icon={Briefcase}>
          <Field label="Professional experience" hint="Paste from your CV — roles, orgs, responsibilities, dates">
            <textarea className={inputCls + " min-h-[140px]"} value={form.experience} onChange={(e) => set("experience", e.target.value)} placeholder="UNICEF SBC & Public Health — Community engagement, risk communication...&#10;AI automation & training — prompt engineering, Make.com..." />
          </Field>
        </Section>

        <Section n="04" title="Skills & goals" icon={Target}>
          <Field label="Skills" hint="Tap to select, add more as needed">
            <div className="flex flex-wrap gap-2">
              {SKILL_TAGS.map((s) => (
                <TagToggle key={s} label={s} active={form.skills.includes(s)} onClick={() => toggleTag("skills", s)} />
              ))}
            </div>
          </Field>
          <Field label="What opportunities are you looking for?">
            <div className="flex flex-wrap gap-2">
              {OPPORTUNITY_TYPES.map((g) => (
                <TagToggle key={g} label={g} active={form.goals.includes(g)} onClick={() => toggleTag("goals", g)} />
              ))}
            </div>
          </Field>
        </Section>

        <Section n="05" title="Preferences" icon={Sliders}>
          <div className="grid grid-cols-1 gap-4">
            <Field label="Work arrangement">
              <div className="relative">
                <select className={inputCls + " appearance-none pr-9"} value={form.remotePref} onChange={(e) => set("remotePref", e.target.value)}>
                  <option>Remote preferred</option>
                  <option>Hybrid okay</option>
                  <option>Onsite okay</option>
                  <option>No preference</option>
                </select>
                <ChevronDown size={15} className="absolute right-3 top-1/2 -translate-y-1/2 text-slate-500 pointer-events-none" />
              </div>
            </Field>
            <Field label="Visa sponsorship needed">
              <div className="relative">
                <select className={inputCls + " appearance-none pr-9"} value={form.visaNeeded} onChange={(e) => set("visaNeeded", e.target.value)}>
                  <option>Not sure</option>
                  <option>Yes, required</option>
                  <option>No, not required</option>
                </select>
                <ChevronDown size={15} className="absolute right-3 top-1/2 -translate-y-1/2 text-slate-500 pointer-events-none" />
              </div>
            </Field>
            <Field label="Funding requirement">
              <div className="relative">
                <select className={inputCls + " appearance-none pr-9"} value={form.funded} onChange={(e) => set("funded", e.target.value)}>
                  <option>Fully funded preferred</option>
                  <option>Partial funding okay</option>
                  <option>Funding not a factor</option>
                </select>
                <ChevronDown size={15} className="absolute right-3 top-1/2 -translate-y-1/2 text-slate-500 pointer-events-none" />
              </div>
            </Field>
            <div className="grid grid-cols-2 gap-4">
              <Field label="Preferred countries" hint="Comma separated"><input className={inputCls} value={form.preferredCountries} onChange={(e) => set("preferredCountries", e.target.value)} placeholder="UK, Canada, Germany" /></Field>
              <Field label="Countries to avoid" hint="Comma separated"><input className={inputCls} value={form.avoidCountries} onChange={(e) => set("avoidCountries", e.target.value)} placeholder="Optional" /></Field>
            </div>
            <Field label="Minimum funding expectation" hint="If applicable — amount, currency, or 'fully covered only'"><input className={inputCls} value={form.minFunding} onChange={(e) => set("minFunding", e.target.value)} /></Field>
            <Field label="Other preferences" hint="Salary floor, org type, time commitment, anything else"><textarea className={inputCls + " min-h-[60px]"} value={form.otherPrefs} onChange={(e) => set("otherPrefs", e.target.value)} /></Field>
          </div>

          <Field label="Opportunity priorities" hint="Ranked highest to lowest — this is what the AI weighs first">
            <div className="space-y-2">
              {priorities.map((p, i) => (
                <div key={p} className="flex items-center gap-3 bg-[#1C222C] border border-[#2A3140] rounded-md px-3 py-2.5">
                  <span className="font-mono text-[11px] text-[#2DD4BF] w-4">{i + 1}</span>
                  <span className="flex-1 text-[14px]">{p}</span>
                  <button type="button" onClick={() => movePriority(i, -1)} disabled={i === 0} className="text-slate-500 hover:text-[#EDEEF0] disabled:opacity-20 px-1">▲</button>
                  <button type="button" onClick={() => movePriority(i, 1)} disabled={i === priorities.length - 1} className="text-slate-500 hover:text-[#EDEEF0] disabled:opacity-20 px-1">▼</button>
                </div>
              ))}
            </div>
          </Field>
        </Section>

        <Section n="06" title="Links" icon={Link2}>
          <Field label="CV link" hint="Google Drive or Dropbox share link"><input className={inputCls} value={form.cvLink} onChange={(e) => set("cvLink", e.target.value)} placeholder="https://drive.google.com/..." /></Field>
          <Field label="LinkedIn"><input className={inputCls} value={form.linkedin} onChange={(e) => set("linkedin", e.target.value)} placeholder="https://linkedin.com/in/..." /></Field>
          <Field label="Portfolio / website"><input className={inputCls} value={form.portfolio} onChange={(e) => set("portfolio", e.target.value)} placeholder="https://..." /></Field>
        </Section>

        {/* Webhook config */}
        <div className="mt-6 p-4 rounded-lg bg-[#1C222C] border border-[#2A3140]">
          <Field label="Make.com webhook URL" hint="Paste your scenario's webhook URL here to send data directly. Leave blank to copy JSON instead.">
            <input className={inputCls} value={form.webhookUrl} onChange={(e) => set("webhookUrl", e.target.value)} placeholder="https://hook.us1.make.com/..." />
          </Field>
        </div>

        <button
          onClick={handleSave}
          className="mt-6 w-full flex items-center justify-center gap-2 bg-[#2DD4BF] text-[#0B0F14] font-semibold py-3.5 rounded-lg hover:bg-[#5EEAD4] transition-colors"
        >
          {status === "sending" ? (
            "Sending…"
          ) : status === "sent" ? (
            <>
              <Check size={17} /> Sent to automation
            </>
          ) : status === "copied" ? (
            <>
              <Copy size={17} /> Copied JSON to clipboard
            </>
          ) : status === "error" ? (
            "Something went wrong — try again"
          ) : form.webhookUrl ? (
            <>
              <Send size={17} /> Save profile
            </>
          ) : (
            <>
              <Copy size={17} /> Copy profile as JSON
            </>
          )}
        </button>
        <p className="text-center font-mono text-[10px] text-slate-500 mt-3">
          Nothing is stored here — refreshing clears the form until it's wired to your automation.
        </p>
      </div>
    </div>
  );
}
