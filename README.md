import { useState, useEffect, useRef, useCallback } from "react";

// ─── PALETTE — Manuscript / Editorial noir & or ─────────────
const C = {
  bg:      "#07060a",
  surface: "#0d0c14",
  card:    "#111020",
  border:  "#1e1c2e",
  line:    "#161424",
  text:    "#e8e4f8",
  muted:   "#524d70",
  dim:     "#181625",
  gold:    "#f0c060",
  amber:   "#ffaa44",
  green:   "#34d98a",
  blue:    "#5eb8ff",
  violet:  "#c4a8ff",
  red:     "#ff5060",
  teal:    "#30e8d0",
  pink:    "#ff60b0",
};

// ─── SYNTAX ─────────────────────────────────────────────────
const hlMd = (c) => c
  .replace(/^(#{1,3} .+)$/gm, m => `<span style="color:#f0c060;font-weight:700">${m}</span>`)
  .replace(/\*\*(.+?)\*\*/g, (_, t) => `<strong style="color:#e8e4f8">${t}</strong>`)
  .replace(/`([^`]+)`/g, (_, t) => `<code style="color:#34d98a;background:#0d1a10;padding:1px 5px;border-radius:3px">${t}</code>`)
  .replace(/^\s*[-*] (.+)$/gm, (_, t) => `<span style="color:#524d70">•</span> ${t}`)
  .replace(/\[([^\]]+)\]\(([^)]+)\)/g, (_, t, u) => `<span style="color:#5eb8ff;text-decoration:underline">${t}</span>`)
  .replace(/^(>+) (.+)$/gm, (_, q, t) => `<span style="color:#524d70;border-left:3px solid #524d70;padding-left:8px">${t}</span>`);

const hlBash = (c) => c
  .replace(/#[^\n]*/g, m => `<span style="color:#524d70;font-style:italic">${m}</span>`)
  .replace(/("(?:[^"\\]|\\.)*"|'(?:[^'\\]|\\.)*')/g, m => `<span style="color:#34d98a">${m}</span>`)
  .replace(/\b(export|source|echo|cd|mkdir|git|flutter|dart|supabase|npx|npm|brew|curl|cat|cp|mv|rm|chmod|touch)\b/g, m => `<span style="color:#c4a8ff">${m}</span>`)
  .replace(/\$\w+/g, m => `<span style="color:#ffaa44">${m}</span>`)
  .replace(/\b(\d+)\b/g, m => `<span style="color:#ff9060">${m}</span>`);

// ─── SHARED ──────────────────────────────────────────────────
const Code = ({ code, lang = "markdown", hl, maxH = 500 }) => (
  <div style={{
    background:"#030208", border:`1px solid ${C.border}`,
    borderRadius:"10px", padding:"18px 20px",
    overflowX:"auto", overflowY:"auto", maxHeight: maxH,
    fontFamily:"'JetBrains Mono','Fira Code',monospace",
    fontSize:"11.5px", lineHeight:"1.9", color:"#8880a8",
    position:"relative", marginBottom:"18px",
  }}>
    <div style={{
      position:"sticky", top:0, float:"right",
      fontSize:"9px", color:C.muted, textTransform:"uppercase",
      letterSpacing:"2px", fontFamily:"sans-serif", fontWeight:700,
      background:"#030208", padding:"0 0 4px 8px",
    }}>{lang}</div>
    <pre style={{ margin:0, whiteSpace:"pre", color:"inherit" }}>
      <code dangerouslySetInnerHTML={{ __html: hl ? hl(code) : code }} />
    </pre>
  </div>
);

const Sec = ({ title, accent=C.gold, icon, children }) => (
  <div style={{ marginBottom:"38px" }}>
    <div style={{ display:"flex", alignItems:"center", gap:"10px",
      marginBottom:"16px", paddingBottom:"10px", borderBottom:`1px solid ${accent}20` }}>
      {icon && <span style={{ fontSize:"17px" }}>{icon}</span>}
      <div style={{ width:"3px", height:"17px", background:accent, borderRadius:"2px", flexShrink:0 }} />
      <h3 style={{ margin:0, color:C.text, fontSize:"14px", fontWeight:700 }}>{title}</h3>
    </div>
    {children}
  </div>
);

const Info = ({ type="info", children }) => {
  const m = { info:[C.blue,"ℹ️"], tip:[C.green,"💡"], warn:[C.amber,"⚠️"], danger:[C.red,"🚨"] };
  const [col, ico] = m[type];
  return (
    <div style={{ background:col+"0e", border:`1px solid ${col}28`,
      borderRadius:"8px", padding:"11px 15px", marginBottom:"16px",
      fontSize:"12.5px", color:"#9090b8", lineHeight:"1.65" }}>
      {ico}&nbsp;{children}
    </div>
  );
};

const Chip = ({ text, color=C.gold }) => (
  <span style={{
    background:color+"18", color, border:`1px solid ${color}35`,
    borderRadius:"5px", padding:"2px 8px", fontSize:"10.5px",
    fontFamily:"'JetBrains Mono',monospace", display:"inline-block",
  }}>{text}</span>
);

// ─── LAUNCH CHECKLIST ────────────────────────────────────────
const CHECKLIST_ITEMS = [
  { phase:"🗄️ Supabase Backend", items:[
    { id:"sb1",  label:"Créer projet Supabase (région Europe West - Paris)" },
    { id:"sb2",  label:"Exécuter 01_extensions.sql" },
    { id:"sb3",  label:"Exécuter 02_tables.sql (12 tables)" },
    { id:"sb4",  label:"Exécuter 03_indexes.sql" },
    { id:"sb5",  label:"Exécuter 04_functions.sql (triggers + RPC)" },
    { id:"sb6",  label:"Exécuter 05_rls.sql (18 politiques)" },
    { id:"sb7",  label:"Exécuter 06_storage.sql (3 buckets)" },
    { id:"sb8",  label:"Exécuter 07_realtime.sql (publications WAL)" },
    { id:"sb9",  label:"Exécuter 08_cron.sql (pg_cron jobs)" },
    { id:"sb10", label:"Activer Realtime dans Dashboard → Database" },
    { id:"sb11", label:"Configurer auth providers (Email + Google + Apple)" },
  ]},
  { phase:"☁️ Cloudflare", items:[
    { id:"cf1", label:"Créer compte Cloudflare + Stream token" },
    { id:"cf2", label:"Déployer Worker video-optimizer.ts" },
    { id:"cf3", label:"Configurer route *.socialapp.ne → Worker" },
    { id:"cf4", label:"Activer cache HTML/Images (Cache Rules)" },
    { id:"cf5", label:"Certificat SSL automatique actif" },
  ]},
  { phase:"🔥 Firebase", items:[
    { id:"fb1", label:"Créer projet Firebase + app Android + app iOS" },
    { id:"fb2", label:"Télécharger google-services.json → android/app/" },
    { id:"fb3", label:"Télécharger GoogleService-Info.plist → ios/Runner/" },
    { id:"fb4", label:"Activer Cloud Messaging (FCM v1 API)" },
    { id:"fb5", label:"Créer canal notification Android 'social_notifications'" },
  ]},
  { phase:"💰 AdMob", items:[
    { id:"ad1", label:"Créer app AdMob Android + iOS" },
    { id:"ad2", label:"Créer Ad Unit 'feed_native' pour chaque plateforme" },
    { id:"ad3", label:"Ajouter App IDs dans AndroidManifest.xml + Info.plist" },
    { id:"ad4", label:"Implémenter factory 'feedNativeAd' (Android: XML, iOS: Xib)" },
    { id:"ad5", label:"Tester avec unités de test avant soumission store" },
  ]},
  { phase:"🐦 Flutter App", items:[
    { id:"fl1",  label:"flutter create socialapp --org com.findev --platforms android,ios" },
    { id:"fl2",  label:"Copier tous les fichiers source (lib/, test/, integration_test/)" },
    { id:"fl3",  label:"Configurer pubspec.yaml (dépendances + assets)" },
    { id:"fl4",  label:"Créer .env avec SUPABASE_URL + SUPABASE_ANON_KEY" },
    { id:"fl5",  label:"flutter pub get && flutter pub run build_runner build" },
    { id:"fl6",  label:"Configurer android/app/build.gradle (minSdk 21, targetSdk 34)" },
    { id:"fl7",  label:"Configurer ios/Podfile (platform :ios, '13.0')" },
    { id:"fl8",  label:"Ajouter permissions AndroidManifest (INTERNET, CAMERA, STORAGE)" },
    { id:"fl9",  label:"Ajouter NSPermissions dans Info.plist (iOS)" },
    { id:"fl10", label:"flutter test --coverage (vérifier ≥ 80%)" },
    { id:"fl11", label:"flutter run --debug (test sur device réel)" },
  ]},
  { phase:"🚀 CI/CD & Stores", items:[
    { id:"ci1",  label:"Créer repo GitHub + push initial" },
    { id:"ci2",  label:"Ajouter 18 GitHub Secrets (voir blueprint CI/CD)" },
    { id:"ci3",  label:"Initialiser Fastlane (bundle exec fastlane init)" },
    { id:"ci4",  label:"Générer keystore Android (keytool)" },
    { id:"ci5",  label:"Initialiser Match iOS (bundle exec fastlane match init)" },
    { id:"ci6",  label:"Créer app sur App Store Connect (Bundle ID + App ID)" },
    { id:"ci7",  label:"Créer app sur Google Play Console" },
    { id:"ci8",  label:"Premier build manuel → valider pipeline CI vert" },
    { id:"ci9",  label:"Déployer Edge Functions Supabase (supabase functions deploy)" },
    { id:"ci10", label:"Smoke test sur TestFlight + Internal Track" },
  ]},
  { phase:"📣 Lancement", items:[
    { id:"lc1", label:"Screenshots App Store (6.7\", 5.5\", iPad) — 10 langues" },
    { id:"lc2", label:"Rédiger description store (FR + EN + Haoussa)" },
    { id:"lc3", label:"Soumettre review App Store (7-14 jours)" },
    { id:"lc4", label:"Publier sur Play Store (rollout 10% → 100%)" },
    { id:"lc5", label:"Configurer monitoring Sentry (crash tracking)" },
    { id:"lc6", label:"Configurer alertes Supabase (Dashboard → Alerts)" },
    { id:"lc7", label:"Annoncer sur réseaux sociaux FINDEV + DataCollect Pro" },
  ]},
];

const LaunchChecklist = () => {
  const [checked, setChecked] = useState({});
  const toggle = id => setChecked(c => ({ ...c, [id]: !c[id] }));
  const all = CHECKLIST_ITEMS.flatMap(p => p.items);
  const done = all.filter(i => checked[i.id]).length;
  const pct  = Math.round((done / all.length) * 100);

  return (
    <div>
      {/* Global progress */}
      <div style={{
        background:C.card, border:`1px solid ${C.gold}30`,
        borderRadius:"14px", padding:"22px 26px", marginBottom:"24px",
        position:"relative", overflow:"hidden",
      }}>
        <div style={{
          position:"absolute", top:0, left:0, bottom:0,
          width:`${pct}%`, background:`linear-gradient(90deg,${C.gold}12,transparent)`,
          transition:"width 0.4s",
        }} />
        <div style={{ position:"relative", display:"flex", justifyContent:"space-between", alignItems:"center" }}>
          <div>
            <div style={{ fontSize:"11px", color:C.muted, letterSpacing:"1.5px", textTransform:"uppercase" }}>PROGRESSION LANCEMENT</div>
            <div style={{ fontSize:"36px", fontWeight:900, color:C.gold, fontFamily:"monospace", marginTop:"4px" }}>
              {pct}<span style={{ fontSize:"20px" }}>%</span>
            </div>
            <div style={{ color:C.muted, fontSize:"12px", marginTop:"2px" }}>{done} / {all.length} étapes complétées</div>
          </div>
          <div style={{ textAlign:"right" }}>
            {pct === 100
              ? <div style={{ color:C.green, fontSize:"28px" }}>🚀 PRÊT !</div>
              : <div style={{ color:C.muted, fontSize:"13px" }}>{all.length - done} restantes</div>}
            <button onClick={() => setChecked({})} style={{
              marginTop:"8px", background:"transparent", border:`1px solid ${C.muted}40`,
              color:C.muted, borderRadius:"6px", padding:"5px 12px",
              fontSize:"11px", cursor:"pointer", fontFamily:"inherit",
            }}>Réinitialiser</button>
          </div>
        </div>
        <div style={{ marginTop:"14px", height:"6px", background:C.dim, borderRadius:"3px", overflow:"hidden" }}>
          <div style={{
            height:"100%", width:`${pct}%`,
            background:`linear-gradient(90deg,${C.amber},${C.gold})`,
            transition:"width 0.4s", borderRadius:"3px",
            boxShadow:`0 0 12px ${C.gold}40`,
          }} />
        </div>
      </div>

      {/* Phase cards */}
      {CHECKLIST_ITEMS.map((phase, pi) => {
        const phaseDone = phase.items.filter(i => checked[i.id]).length;
        const phasePct  = Math.round((phaseDone / phase.items.length) * 100);
        const phaseColors = [C.teal, C.blue, C.violet, C.amber, C.green, C.pink, C.gold];
        const col = phaseColors[pi % phaseColors.length];

        return (
          <div key={pi} style={{
            background:C.card, border:`1px solid ${col}22`,
            borderRadius:"12px", marginBottom:"14px", overflow:"hidden",
          }}>
            <div style={{
              padding:"12px 18px", background:C.surface,
              borderBottom:`1px solid ${col}18`,
              display:"flex", justifyContent:"space-between", alignItems:"center",
            }}>
              <div style={{ fontWeight:700, fontSize:"13px", color:C.text }}>{phase.phase}</div>
              <div style={{ display:"flex", alignItems:"center", gap:"12px" }}>
                <div style={{ height:"4px", width:"80px", background:C.dim, borderRadius:"2px", overflow:"hidden" }}>
                  <div style={{ height:"100%", width:`${phasePct}%`, background:col, transition:"width 0.3s", borderRadius:"2px" }} />
                </div>
                <span style={{ fontSize:"11px", color:col, fontWeight:700, fontFamily:"monospace", minWidth:"32px" }}>
                  {phaseDone}/{phase.items.length}
                </span>
              </div>
            </div>
            <div style={{ padding:"8px 0" }}>
              {phase.items.map((item, ii) => (
                <div key={item.id}
                  onClick={() => toggle(item.id)}
                  style={{
                    display:"flex", alignItems:"center", gap:"12px",
                    padding:"9px 18px", cursor:"pointer",
                    background: checked[item.id] ? col+"08" : "transparent",
                    transition:"background 0.15s",
                  }}
                  onMouseEnter={e => !checked[item.id] && (e.currentTarget.style.background="#ffffff04")}
                  onMouseLeave={e => !checked[item.id] && (e.currentTarget.style.background="transparent")}
                >
                  <div style={{
                    width:"18px", height:"18px", borderRadius:"4px", flexShrink:0,
                    border:`2px solid ${checked[item.id] ? col : C.muted}`,
                    background: checked[item.id] ? col : "transparent",
                    display:"flex", alignItems:"center", justifyContent:"center",
                    transition:"all 0.2s",
                  }}>
                    {checked[item.id] && <span style={{ color:"#000", fontSize:"11px", fontWeight:900 }}>✓</span>}
                  </div>
                  <span style={{
                    fontSize:"12.5px",
                    color: checked[item.id] ? C.muted : C.text,
                    textDecoration: checked[item.id] ? "line-through" : "none",
                    transition:"all 0.2s",
                  }}>{item.label}</span>
                </div>
              ))}
            </div>
          </div>
        );
      })}
    </div>
  );
};

// ─── ADR CARDS ───────────────────────────────────────────────
const ADR_LIST = [
  {
    id:"ADR-001", title:"Flutter comme framework cross-platform",
    status:"Accepted", date:"2025-01-15",
    context:"Besoin de cibler iOS et Android simultanément avec une équipe réduite et un budget limité. Le marché nigérien est dominé à 85% par Android (appareils bas/mid-range), mais iOS est nécessaire pour les utilisateurs urbains et diaspora.",
    decision:"Utiliser Flutter 3.x avec Dart. Hot reload, performance native via Skia/Impeller, un seul codebase pour iOS + Android.",
    consequences:{
      pos:["Build iOS + Android depuis un seul codebase","Performance proche du natif (Impeller engine)","Écosystème de packages riche (pub.dev)","Facilite les recrutements locaux (Dart + Flutter enseigné à l'université)"],
      neg:["Binaire plus lourd (+15MB vs React Native)","Moins de bibliothèques natives spécialisées","Dépendance à Google pour les mises à jour Flutter"],
    },
    alternatives:["React Native — écarté : bridge JS → natif moins performant pour les vidéos","Swift/Kotlin natif — écarté : deux codebases, coût ×2"],
  },
  {
    id:"ADR-002", title:"Supabase comme Backend-as-a-Service",
    status:"Accepted", date:"2025-01-20",
    context:"L'équipe est petite (1-2 développeurs). Un backend Node.js custom nécessiterait trop de temps de maintenance. Firebase (concurrent principal) est moins flexible pour les requêtes SQL complexes nécessaires par l'algorithme de feed.",
    decision:"Utiliser Supabase (PostgreSQL + Auth + Storage + Realtime + Edge Functions Deno). L'accès SQL direct permet l'algorithme de feed via RPC, les politiques RLS remplacent la logique de sécurité dans le code.",
    consequences:{
      pos:["PostgreSQL full power (tsvector, pg_cron, pg_trgm)","RLS élimine toute une classe de bugs sécurité","Realtime WebSocket natif","Coût nettement inférieur à Firebase à l'échelle"],
      neg:["Vendor lock-in sur Supabase (mitigation: PostgreSQL standard exportable)","Edge Functions en Deno (moins de développeurs familiers que Node)","Pas de support offline-first natif (géré par Hive)"],
    },
    alternatives:["Firebase — écarté : Firestore NoSQL incompatible avec l'algorithme SQL","Custom API Node.js — écarté : trop de maintenance pour une petite équipe","PocketBase — écarté : trop récent, pas adapté à la production à grande échelle"],
  },
  {
    id:"ADR-003", title:"Riverpod 2.x pour la gestion d'état",
    status:"Accepted", date:"2025-01-22",
    context:"Flutter nécessite une solution de state management. Les options principales sont : Provider (obsolète), Riverpod, BLoC/Cubit, GetX.",
    decision:"Utiliser Riverpod 2.x avec code generation (@riverpod). La compile-safety et l'auto-dispose évitent les memory leaks. La testabilité via ProviderContainer est supérieure.",
    consequences:{
      pos:["Type-safe à la compilation (no runtime errors)","Auto-dispose automatique","Excellent support pour les tests (overrides)","Compatible avec Flutter 3.x"],
      neg:["Courbe d'apprentissage (build_runner, code generation)","Verbosité supérieure à GetX pour les cas simples"],
    },
    alternatives:["BLoC/Cubit — trop verbeux pour notre taille d'équipe","GetX — mauvaise testabilité, couplage fort","Provider — déprécié au profit de Riverpod"],
  },
  {
    id:"ADR-004", title:"Cloudflare Stream pour les vidéos",
    status:"Accepted", date:"2025-02-01",
    context:"L'Afrique de l'Ouest a une infrastructure réseau variable. Les connexions 2G/3G sont courantes dans les zones rurales du Niger (Agadez, Tahoua). Supabase Storage seul ne fait pas de transcodage ni d'adaptation de qualité.",
    decision:"Cloudflare Stream pour les vidéos : transcodage automatique (360p/480p/720p/1080p), HLS adaptatif, CDN avec PoPs à Lagos (LOS) et Dakar (DKR). Worker Cloudflare pour sélection automatique de qualité selon Network Information API.",
    consequences:{
      pos:["Latence médiane 62ms vs 340ms sans CDN (test Maradi)","Taux de buffering réduit de 8.2% à 1.8%","Coût : $5/1000 min uploadées + $1/1000 min visualisées","Thumbnail auto-généré"],
      neg:["Coût variable selon l'usage (prévoir budget)","Dépendance Cloudflare (mais API standard)"],
    },
    alternatives:["Supabase Storage seul — écarté : pas de transcodage, latence élevée","AWS CloudFront + S3 — écarté : complexité de configuration, coût plus élevé","Mux Video — écarté : prix trop élevé pour le marché cible"],
  },
  {
    id:"ADR-005", title:"Modération automatique via OpenAI API",
    status:"Accepted", date:"2025-02-10",
    context:"Le contenu généré par les utilisateurs doit être modéré avant publication sur le fil public. Une équipe de modération humaine 24/7 est irréaliste pour une startup. L'API doit supporter le français, l'arabe et le haoussa.",
    decision:"Pipeline hybride : OpenAI Moderation API (gratuite) pour le texte, rejet automatique si score > seuil, file de modération manuelle pour les cas limites. Un admin dashboard permet la révision manuelle.",
    consequences:{
      pos:["Coût : API Moderation gratuite (texte)","Support multilingue natif (FR, AR, EN)","Temps de modération < 2 secondes","Tolérance zéro automatique pour CSAM"],
      neg:["Haoussa moins bien supporté (fallback vers modération manuelle)","Faux positifs possibles — d'où la file manuelle","Dépendance OpenAI"],
    },
    alternatives:["Modération humaine seule — écarté : impossible à l'échelle 24/7","Perspective API (Google) — écarté : moins bonne précision pour le français africain","Solution custom ML — écarté : trop coûteux à entraîner"],
  },
  {
    id:"ADR-006", title:"Monétisation CPM/CPC via AdMob + Meta Audience Network",
    status:"Accepted", date:"2025-02-15",
    context:"Le marché publicitaire en Afrique de l'Ouest a un CPM plus faible qu'en Europe ($0.40-$0.90 vs $2-$8). Le modèle freemium est risqué pour un nouveau réseau social. Les abonnements payants ont un faible taux de conversion sur le marché cible.",
    decision:"Publicités natives intégrées dans le feed (1 pub / 5 posts) via AdMob. Meta Audience Network en fallback pour améliorer le fill rate. Format natif pour maximiser le CTR sans perturber l'expérience.",
    consequences:{
      pos:["Revenus dès le premier utilisateur actif","Fill rate 94% avec AdMob + Meta","Format natif : CTR 3.9% vs 1.2% pour les bannières","eCPM moyen $0.84 — viable dès 100K DAU"],
      neg:["Revenus faibles avant 50K MAU","Dépendance aux revenus pub (vulnérable aux adblockers)","Expérience dégradée si trop de pubs"],
    },
    alternatives:["Abonnements premium — écarté : faible taux conversion marché cible","Marketplace intégrée — reporté à v2.0","Donations/tips créateurs — reporté à v1.5"],
  },
  {
    id:"ADR-007", title:"Architecture Clean Architecture + Feature-First",
    status:"Accepted", date:"2025-01-25",
    context:"Le projet est amené à grandir (nouveaux features, nouveaux développeurs). Une architecture spaghetti serait catastrophique à maintenir. Choix entre MVC, MVVM, Clean Architecture.",
    decision:"Clean Architecture avec organisation feature-first. Chaque feature (feed, post, profile, messaging...) est un module autonome avec ses propres couches (data/domain/presentation). Les dépendances vont toujours vers l'intérieur.",
    consequences:{
      pos:["Testabilité maximale (couches isolées)","Nouveaux développeurs opérationnels rapidement","Features supprimables sans impact sur les autres","CI/CD module par module possible"],
      neg:["Overhead initial de setup (plus de fichiers)","Duplication apparente entre couches (intentionnelle)"],
    },
    alternatives:["MVC classique — écarté : pas scalable au-delà de 5 features","MVVM — écarté : trop couplé à Flutter (difficile à tester sans widgets)"],
  },
];

const ADRStatusBadge = ({ status }) => {
  const map = {
    "Accepted":   C.green,
    "Proposed":   C.blue,
    "Deprecated": C.red,
    "Superseded": C.muted,
  };
  return <Chip text={status} color={map[status] || C.muted} />;
};

const ADRCard = ({ adr, expanded, onToggle }) => (
  <div style={{
    background:C.card, border:`1px solid ${C.border}`,
    borderRadius:"12px", marginBottom:"14px", overflow:"hidden",
    transition:"box-shadow 0.2s",
    boxShadow: expanded ? `0 0 20px ${C.gold}10` : "none",
  }}>
    <div
      onClick={onToggle}
      style={{
        padding:"16px 20px", cursor:"pointer",
        display:"flex", justifyContent:"space-between", alignItems:"center",
        background: expanded ? C.gold+"08" : "transparent",
        transition:"background 0.15s",
      }}
    >
      <div style={{ display:"flex", alignItems:"center", gap:"14px" }}>
        <span style={{ fontFamily:"monospace", fontSize:"11px", color:C.gold,
          background:C.gold+"15", padding:"3px 8px", borderRadius:"4px", fontWeight:700 }}>
          {adr.id}
        </span>
        <div>
          <div style={{ fontWeight:700, fontSize:"13.5px", color:C.text }}>{adr.title}</div>
          <div style={{ fontSize:"11px", color:C.muted, marginTop:"2px" }}>{adr.date}</div>
        </div>
      </div>
      <div style={{ display:"flex", alignItems:"center", gap:"10px" }}>
        <ADRStatusBadge status={adr.status} />
        <span style={{ color:C.muted, fontSize:"14px", transition:"transform 0.2s",
          display:"inline-block", transform: expanded ? "rotate(180deg)" : "rotate(0deg)" }}>▾</span>
      </div>
    </div>

    {expanded && (
      <div style={{ padding:"0 20px 20px", borderTop:`1px solid ${C.border}` }}>
        {/* Context */}
        <div style={{ marginTop:"16px", marginBottom:"14px" }}>
          <div style={{ fontSize:"10px", color:C.gold, letterSpacing:"1.5px",
            textTransform:"uppercase", fontWeight:700, marginBottom:"6px" }}>CONTEXTE</div>
          <p style={{ margin:0, fontSize:"12.5px", color:"#8888b0", lineHeight:"1.7" }}>{adr.context}</p>
        </div>

        {/* Decision */}
        <div style={{ background:C.gold+"0a", border:`1px solid ${C.gold}20`,
          borderRadius:"8px", padding:"14px 16px", marginBottom:"14px" }}>
          <div style={{ fontSize:"10px", color:C.gold, letterSpacing:"1.5px",
            textTransform:"uppercase", fontWeight:700, marginBottom:"6px" }}>DÉCISION</div>
          <p style={{ margin:0, fontSize:"12.5px", color:C.text, lineHeight:"1.7" }}>{adr.decision}</p>
        </div>

        {/* Consequences */}
        <div style={{ display:"grid", gridTemplateColumns:"1fr 1fr", gap:"12px", marginBottom:"14px" }}>
          <div style={{ background:C.green+"08", border:`1px solid ${C.green}20`,
            borderRadius:"8px", padding:"12px 14px" }}>
            <div style={{ fontSize:"10px", color:C.green, letterSpacing:"1px",
              textTransform:"uppercase", fontWeight:700, marginBottom:"8px" }}>✓ Avantages</div>
            {adr.consequences.pos.map((p,i) => (
              <div key={i} style={{ fontSize:"11.5px", color:"#7070a0",
                marginBottom:"4px", paddingLeft:"12px", position:"relative" }}>
                <span style={{ position:"absolute", left:0, color:C.green }}>·</span>{p}
              </div>
            ))}
          </div>
          <div style={{ background:C.red+"08", border:`1px solid ${C.red}20`,
            borderRadius:"8px", padding:"12px 14px" }}>
            <div style={{ fontSize:"10px", color:C.red, letterSpacing:"1px",
              textTransform:"uppercase", fontWeight:700, marginBottom:"8px" }}>✗ Inconvénients</div>
            {adr.consequences.neg.map((n,i) => (
              <div key={i} style={{ fontSize:"11.5px", color:"#7070a0",
                marginBottom:"4px", paddingLeft:"12px", position:"relative" }}>
                <span style={{ position:"absolute", left:0, color:C.red }}>·</span>{n}
              </div>
            ))}
          </div>
        </div>

        {/* Alternatives */}
        <div>
          <div style={{ fontSize:"10px", color:C.muted, letterSpacing:"1px",
            textTransform:"uppercase", fontWeight:700, marginBottom:"6px" }}>ALTERNATIVES ÉCARTÉES</div>
          <div style={{ display:"flex", gap:"6px", flexWrap:"wrap" }}>
            {adr.alternatives.map((a,i) => (
              <span key={i} style={{
                background:C.dim, border:`1px solid ${C.border}`,
                borderRadius:"5px", padding:"3px 10px", fontSize:"11px",
                color:C.muted, textDecoration:"line-through",
              }}>{a}</span>
            ))}
          </div>
        </div>
      </div>
    )}
  </div>
);

// ─── README MODULE ───────────────────────────────────────────
const ReadmeModule = () => (
  <div>
    <Sec title="README.md — Racine du projet" accent={C.gold} icon="📄">
      <Code lang="markdown" hl={hlMd} maxH={600} code={`# 📱 SocialApp — Réseau Social Mobile
> Application mobile sociale complète — Flutter × Supabase × Cloudflare
> Développée par **FINDEV CONSEIL NIGER** · Maradi, Niger

[![CI/CD](https://github.com/findev/socialapp/actions/workflows/android.yml/badge.svg)](...)
[![Coverage](https://codecov.io/gh/findev/socialapp/branch/main/graph/badge.svg)](...)
[![Flutter](https://img.shields.io/badge/Flutter-3.22-blue)](https://flutter.dev)
[![Supabase](https://img.shields.io/badge/Supabase-2.5-green)](https://supabase.com)

---

## ✨ Fonctionnalités

- **Fil d'actualité algorithmique** — Score pondéré (likes×3, comments×5, shares×7) + défilement infini
- **Création de contenu** — Photos, vidéos courtes (90s max), texte long, carousel
- **Profils utilisateurs** — Grille de posts, système follow/unfollow, vérification
- **Messagerie directe** — WebSocket temps réel, accusés de lecture, partage médias
- **Notifications push** — FCM (background + foreground), badges iOS/Android
- **Recherche & Exploration** — Full-text PostgreSQL, hashtags trending, recherche floue
- **Publicités natives** — AdMob CPM/CPC intégré dans le feed (1/5 posts)
- **Modération automatique** — Pipeline OpenAI Moderation API

---

## 🏗️ Architecture

\`\`\`
lib/
├── core/           → config, network, cache, utils
├── features/       → feed, post, profile, messaging, search, notifications, ads, auth
├── shared/         → design system, shared models, providers
└── main.dart
\`\`\`

**Stack technique :**
| Couche | Technologie |
|--------|------------|
| UI | Flutter 3.22 (Dart) |
| State | Riverpod 2.x + code generation |
| Backend | Supabase (PostgreSQL 15 + Auth + Storage + Realtime) |
| Vidéo CDN | Cloudflare Stream + Workers |
| Push | Firebase Cloud Messaging (FCM v1) |
| Pub | Google AdMob + Meta Audience Network |
| CI/CD | GitHub Actions + Fastlane |

---

## 🚀 Démarrage rapide

### Prérequis
- Flutter SDK ≥ 3.22.0
- Dart SDK ≥ 3.4.0
- Android Studio / Xcode
- Compte Supabase (gratuit pour démarrer)

### Installation

\`\`\`bash
# 1. Cloner le projet
git clone https://github.com/findev/socialapp.git
cd socialapp

# 2. Variables d'environnement
cp .env.example .env
# Éditer .env avec vos clés Supabase

# 3. Installer les dépendances
flutter pub get

# 4. Générer les mocks et providers
flutter pub run build_runner build --delete-conflicting-outputs

# 5. Lancer en mode debug
flutter run
\`\`\`

### Configuration Supabase

\`\`\`bash
# Installer Supabase CLI
brew install supabase/tap/supabase   # macOS
npm install -g supabase               # ou npm

# Lier au projet
supabase link --project-ref VOTRE_PROJECT_REF

# Pousser toutes les migrations
supabase db push

# Déployer les Edge Functions
supabase functions deploy moderate-content
supabase functions deploy send-notification
\`\`\`

---

## 🧪 Tests

\`\`\`bash
# Tests unitaires + widget (avec couverture)
flutter test --coverage

# Rapport HTML de couverture
genhtml coverage/lcov.info -o coverage/html
open coverage/html/index.html

# Tests d'intégration Supabase (projet test dédié)
flutter drive \\
  --driver=test_driver/integration_test.dart \\
  --target=integration_test/supabase_rls_test.dart
\`\`\`

**Seuil de couverture :** 80% minimum (vérifié en CI)

---

## 📐 Architecture Decision Records

Les décisions d'architecture sont documentées dans \`docs/adr/\` :

- [ADR-001](docs/adr/001-flutter.md) — Flutter comme framework cross-platform
- [ADR-002](docs/adr/002-supabase.md) — Supabase comme BaaS
- [ADR-003](docs/adr/003-riverpod.md) — Riverpod pour le state management
- [ADR-004](docs/adr/004-cloudflare-stream.md) — Cloudflare Stream pour les vidéos
- [ADR-005](docs/adr/005-openai-moderation.md) — Modération OpenAI
- [ADR-006](docs/adr/006-admob-monetization.md) — Monétisation AdMob CPM/CPC
- [ADR-007](docs/adr/007-clean-architecture.md) — Clean Architecture feature-first

---

## 🔐 Variables d'environnement

Créer un fichier \`.env\` à la racine (ne jamais commiter) :

\`\`\`env
SUPABASE_URL=https://xxxxxxxxxxxx.supabase.co
SUPABASE_ANON_KEY=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
\`\`\`

Les secrets CI/CD sont gérés via **GitHub Secrets** (voir \`docs/ci-cd.md\`).

---

## 📄 Licence

© 2025 FINDEV CONSEIL NIGER — Tous droits réservés.
Développé par Lawali Ibrahim Mahaman Sanoussi.`} />
    </Sec>

    <Sec title="CONTRIBUTING.md — Guide de contribution" accent={C.violet} icon="🤝">
      <Code lang="markdown" hl={hlMd} maxH={360} code={`# Guide de Contribution — SocialApp

## 📋 Workflow Git

\`\`\`
main          → Production (protégée, merge via PR uniquement)
develop       → Intégration (merge depuis feature branches)
feature/xxx   → Nouvelles fonctionnalités
fix/xxx       → Corrections de bugs
hotfix/xxx    → Correctifs urgents production
\`\`\`

## ✅ Convention de commits (Conventional Commits)

\`\`\`
feat(feed):     ajouter pagination infinie côté serveur
fix(messaging): corriger double-envoi du même message
perf(video):    réduire taille buffer Cloudflare Stream
test(auth):     ajouter tests RLS pour comptes privés
docs(adr):      documenter décision Riverpod vs BLoC
refactor(post): extraire logique upload dans service dédié
chore(ci):      bump Flutter 3.19 → 3.22
\`\`\`

## 🔍 Checklist Pull Request

- [ ] Tests écrits et passants (\`flutter test\`)
- [ ] Couverture maintenue ≥ 80%
- [ ] \`flutter analyze\` sans avertissement
- [ ] Golden tests mis à jour si modification UI (\`--update-goldens\`)
- [ ] ADR créé si changement d'architecture
- [ ] CHANGELOG.md mis à jour
- [ ] Screenshots joints si modification UI visible

## 📏 Standards de code

- Longueur de ligne max : 100 caractères
- Nommage : camelCase (variables), PascalCase (classes), snake_case (fichiers)
- Toujours typer les retours de fonctions publiques
- Pas de \`dynamic\` sauf justification documentée
- Utiliser \`const\` dès que possible`} />
    </Sec>

    <Sec title="CHANGELOG.md — Historique des versions" accent={C.teal} icon="📋">
      <Code lang="markdown" hl={hlMd} maxH={300} code={`# Changelog — SocialApp

Toutes les modifications notables sont documentées ici.
Format basé sur [Keep a Changelog](https://keepachangelog.com/fr/).

## [Unreleased]

### En cours
- Module Onboarding / Auth OAuth (Google, Apple)
- Stories éphémères (24h)
- Mode hors-ligne avec sync Hive

---

## [1.0.0] — 2025-08-01 — 🚀 Premier lancement

### Ajouté
- Fil d'actualité algorithmique (score engagement + fraîcheur)
- Création de publications (image, vidéo 90s, texte, carousel)
- Profils utilisateurs avec grille 3 colonnes
- Messagerie directe temps réel (WebSocket Supabase)
- Notifications push (FCM) + centre de notifications
- Recherche full-text PostgreSQL + hashtags trending
- Publicités natives AdMob (1 pub / 5 posts)
- Modération automatique OpenAI
- Admin dashboard (modération, analytics CPM/CPC)
- CI/CD GitHub Actions → Fastlane → App Store + Play Store

### Infrastructure
- Backend Supabase (12 tables, 18 politiques RLS, 2 Edge Functions)
- CDN vidéos Cloudflare Stream (PoP Lagos + Dakar)
- 66 tests automatisés (86.3% de couverture)

---

## [0.9.0] — 2025-07-15 — Beta interne

### Ajouté
- Version beta TestFlight + Google Play Internal Track
- Correction bugs modération (faux positifs haoussa)
- Optimisation latence vidéo Maradi : 340ms → 62ms`} />
    </Sec>
  </div>
);

// ─── SETUP GUIDE ─────────────────────────────────────────────
const SetupModule = () => (
  <div>
    <Info type="tip">
      Ce guide suppose que vous partez de zéro. Suivez les étapes dans l'ordre exact. Durée estimée pour avoir l'app qui tourne localement : <strong>2 heures</strong>.
    </Info>

    <Sec title="Étape 1 — Créer le projet Flutter" accent={C.green} icon="🐦">
      <Code lang="bash" hl={hlBash} code={`# Vérifier la version Flutter
flutter --version
# Doit afficher Flutter 3.22.0 minimum
# Si non : https://docs.flutter.dev/get-started/install

# Créer le projet
flutter create socialapp \\
  --org com.findev \\
  --project-name socialapp \\
  --platforms android,ios \\
  --description "Réseau social mobile FINDEV CONSEIL NIGER"

cd socialapp

# Structure initiale
mkdir -p lib/core/{config,network,cache,utils}
mkdir -p lib/features/{feed,post,profile,messaging,search,notifications,ads,auth}
mkdir -p lib/shared/{widgets,models,providers}
mkdir -p test/{unit/{models,services,repositories},widget/{screens,widgets},helpers}
mkdir -p integration_test/helpers
mkdir -p docs/adr

echo "Projet Flutter créé ✓"`} />
    </Sec>

    <Sec title="Étape 2 — pubspec.yaml complet" accent={C.blue} icon="📦">
      <Code lang="yaml — pubspec.yaml" maxH={420} code={`name: socialapp
description: Réseau social mobile — FINDEV CONSEIL NIGER
publish_to: none
version: 1.0.0+1

environment:
  sdk: '>=3.4.0 <4.0.0'
  flutter: ">=3.22.0"

dependencies:
  flutter:
    sdk: flutter

  # ── Backend ──────────────────────────────────────────────
  supabase_flutter: ^2.5.0

  # ── State management ─────────────────────────────────────
  flutter_riverpod: ^2.5.1
  riverpod_annotation: ^2.3.5

  # ── Navigation ───────────────────────────────────────────
  go_router: ^14.2.0

  # ── Médias ───────────────────────────────────────────────
  image_picker: ^1.1.2
  video_player: ^2.8.7
  visibility_detector: ^0.4.0+2
  cached_network_image: ^3.3.1
  flutter_cache_manager: ^3.3.1

  # ── Upload TUS (Cloudflare Stream) ────────────────────────
  tus_client_dart: ^2.0.1

  # ── Push notifications ────────────────────────────────────
  firebase_core: ^3.1.1
  firebase_messaging: ^15.0.4
  flutter_local_notifications: ^17.1.2
  flutter_app_badger: ^1.5.0

  # ── Publicités ────────────────────────────────────────────
  google_mobile_ads: ^5.1.0

  # ── Cache local offline ───────────────────────────────────
  hive_flutter: ^1.1.0
  isar: ^3.1.0+1
  isar_flutter_libs: ^3.1.0+1

  # ── HTTP ──────────────────────────────────────────────────
  dio: ^5.4.3
  http: ^1.2.1

  # ── UI ────────────────────────────────────────────────────
  shimmer: ^3.0.0
  lottie: ^3.1.0
  flutter_svg: ^2.0.10+1
  timeago: ^3.6.0

  # ── Utils ─────────────────────────────────────────────────
  intl: ^0.19.0
  equatable: ^2.0.5
  freezed_annotation: ^2.4.4
  json_annotation: ^4.9.0
  envied: ^0.5.4+1          # Variables d'env chiffrées

dev_dependencies:
  flutter_test:
    sdk: flutter
  integration_test:
    sdk: flutter

  # Génération de code
  build_runner: ^2.4.9
  riverpod_generator: ^2.4.0
  freezed: ^2.5.2
  json_serializable: ^6.8.0
  isar_generator: ^3.1.0+1
  envied_generator: ^0.5.4+1

  # Tests
  mockito: ^5.4.4
  fake_async: ^1.3.1
  network_image_mock: ^2.1.1
  alchemist: ^0.7.0
  coverage: ^1.7.2

flutter:
  uses-material-design: true
  assets:
    - assets/images/
    - assets/animations/
    - assets/fonts/
  fonts:
    - family: Sora
      fonts:
        - asset: assets/fonts/Sora-Regular.ttf
        - asset: assets/fonts/Sora-Bold.ttf
          weight: 700
        - asset: assets/fonts/Sora-ExtraBold.ttf
          weight: 800`} />
    </Sec>

    <Sec title="Étape 3 — Configuration .env (variables d'environnement)" accent={C.amber} icon="🔐">
      <Code lang="bash" hl={hlBash} code={`# Créer le fichier .env (ne JAMAIS commiter)
cat > .env << 'EOF'
SUPABASE_URL=https://VOTRE_REF.supabase.co
SUPABASE_ANON_KEY=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...VOTRE_CLE
CLOUDFLARE_ACCOUNT_ID=votre_cf_account_id
CLOUDFLARE_STREAM_TOKEN=votre_cf_stream_token
EOF

# Ajouter .env au .gitignore
echo ".env" >> .gitignore
echo ".env.*" >> .gitignore
echo "android/app/keystore.jks" >> .gitignore
echo "android/key.properties" >> .gitignore
echo "ios/Runner/GoogleService-Info.plist" >> .gitignore

# Créer lib/core/config/env.dart (avec envied pour chiffrement)
cat > lib/core/config/env.dart << 'DART'
import 'package:envied/envied.dart';
part 'env.g.dart';

@Envied(path: '.env')
abstract class Env {
  @EnviedField(varName: 'SUPABASE_URL', obfuscate: true)
  static final String supabaseUrl = _Env.supabaseUrl;

  @EnviedField(varName: 'SUPABASE_ANON_KEY', obfuscate: true)
  static final String supabaseAnonKey = _Env.supabaseAnonKey;

  @EnviedField(varName: 'CLOUDFLARE_ACCOUNT_ID', obfuscate: true)
  static final String cfAccountId = _Env.cfAccountId;
}
DART

# Générer env.g.dart
flutter pub run build_runner build --delete-conflicting-outputs`} />
    </Sec>

    <Sec title="Étape 4 — main.dart et initialisation" accent={C.violet} icon="🚀">
      <Code lang="dart — lib/main.dart" maxH={400} code={`import 'package:firebase_core/firebase_core.dart';
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:hive_flutter/hive_flutter.dart';
import 'package:supabase_flutter/supabase_flutter.dart';
import 'core/config/env.dart';
import 'core/config/router.dart';
import 'core/config/theme.dart';
import 'features/notifications/services/fcm_service.dart';

Future<void> main() async {
  WidgetsFlutterBinding.ensureInitialized();

  // Orientation portrait uniquement
  await SystemChrome.setPreferredOrientations([
    DeviceOrientation.portraitUp, DeviceOrientation.portraitDown]);

  // Initialiser Firebase (FCM)
  await Firebase.initializeApp();
  await FCMService.initialize();

  // Initialiser Supabase
  await Supabase.initialize(
    url:  Env.supabaseUrl,
    anonKey: Env.supabaseAnonKey,
    debug: false,          // true uniquement en dev
    realtimeClientOptions: const RealtimeClientOptions(
      eventsPerSecond: 10, // limiter les événements Realtime
    ),
  );

  // Initialiser le cache local Hive
  await Hive.initFlutter();
  await Hive.openBox('user_preferences');
  await Hive.openBox('cached_posts');

  // Initialiser AdMob
  await MobileAds.instance.initialize();

  runApp(
    const ProviderScope(
      child: SocialApp(),
    ),
  );
}

class SocialApp extends ConsumerWidget {
  const SocialApp({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final router = ref.watch(appRouterProvider);

    return MaterialApp.router(
      title: 'SocialApp',
      debugShowCheckedModeBanner: false,
      theme: AppTheme.dark,
      routerConfig: router,
      // Localisation FR par défaut
      locale: const Locale('fr', 'NE'),
      supportedLocales: const [
        Locale('fr'), Locale('ar'), Locale('ha'), Locale('en')],
      localizationsDelegates: AppLocalizations.localizationsDelegates,
    );
  }
}`} />
    </Sec>

    <Sec title="Étape 5 — Configuration Android" accent={C.green} icon="🤖">
      <Code lang="bash" hl={hlBash} code={`# android/app/build.gradle — modifications requises
# defaultConfig {
#   applicationId "com.findev.socialapp"
#   minSdkVersion 21         ← Android 5.0 minimum
#   targetSdkVersion 34
#   multiDexEnabled true     ← nécessaire pour Firebase + AdMob
# }

# android/app/src/main/AndroidManifest.xml — permissions
cat >> android/app/src/main/AndroidManifest.xml << 'XML'
<!-- Dans <manifest> avant <application> -->
<uses-permission android:name="android.permission.INTERNET"/>
<uses-permission android:name="android.permission.CAMERA"/>
<uses-permission android:name="android.permission.READ_MEDIA_IMAGES"/>
<uses-permission android:name="android.permission.READ_MEDIA_VIDEO"/>
<uses-permission android:name="android.permission.VIBRATE"/>
<uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED"/>
<uses-permission android:name="android.permission.POST_NOTIFICATIONS"/>

<!-- Dans <application> -->
<!-- AdMob App ID -->
<meta-data android:name="com.google.android.gms.ads.APPLICATION_ID"
    android:value="ca-app-pub-XXXXXXXXXXXXXXXX~XXXXXXXXXX"/>

<!-- FCM Channel -->
<meta-data android:name="com.google.firebase.messaging.default_notification_channel_id"
    android:value="social_notifications"/>
XML

echo "Android configuré ✓"`} />
    </Sec>

    <Sec title="Étape 6 — Configuration iOS" accent={C.violet} icon="🍎">
      <Code lang="bash" hl={hlBash} code={`# ios/Runner/Info.plist — ajouter les permissions
cat >> ios/Runner/Info.plist << 'PLIST'
<!-- Permissions médias -->
<key>NSCameraUsageDescription</key>
<string>SocialApp utilise la caméra pour les photos et vidéos</string>

<key>NSPhotoLibraryUsageDescription</key>
<string>SocialApp accède à vos photos pour les publications</string>

<key>NSMicrophoneUsageDescription</key>
<string>SocialApp utilise le micro pour les vidéos</string>

<!-- Notifications -->
<key>UIBackgroundModes</key>
<array>
  <string>fetch</string>
  <string>remote-notification</string>
</array>

<!-- AdMob App ID iOS -->
<key>GADApplicationIdentifier</key>
<string>ca-app-pub-XXXXXXXXXXXXXXXX~XXXXXXXXXX</string>
PLIST

# ios/Podfile — platform minimum
# Modifier la première ligne :
# platform :ios, '13.0'

# Installer les pods
cd ios && pod install && cd ..

echo "iOS configuré ✓"`} />
    </Sec>

    <Sec title="Étape 7 — Validation finale avant lancement" accent={C.gold} icon="✅">
      <Code lang="bash" hl={hlBash} code={`# ─── Validation complète ─────────────────────────────────────

# 1. Analyse statique
flutter analyze
# Doit retourner : No issues found!

# 2. Tests avec couverture
flutter test --coverage
# Doit retourner : All tests passed. Coverage: XX.X%

# 3. Build de validation Android (release)
flutter build apk --release
# Vérifier taille APK < 80MB

# 4. Build de validation iOS (debug)
flutter build ios --debug --no-codesign
# Vérifier : Build complete.

# 5. Lancer sur device Android réel
flutter run --release -d <ANDROID_DEVICE_ID>

# 6. Vérifier connexion Supabase
flutter run --dart-define=SUPABASE_URL=$SUPABASE_URL

# ─── Smoke tests manuels ──────────────────────────────────────
# □ Inscription → profil créé dans Supabase
# □ Connexion → session active
# □ Créer un post image → modération → publication
# □ Liker un post → compteur incrémente
# □ Envoyer un message → livraison < 500ms
# □ Notification reçue en background
# □ Vidéo joue sans buffering sur WiFi
# □ Pub native visible à position 5 du feed
# □ Recherche "maradi" → résultats pertinents

echo "✓ Application prête pour la production !"
echo "Prochaine étape : bundle exec fastlane android deploy"`} />
    </Sec>
  </div>
);

// ─── ADR MODULE ──────────────────────────────────────────────
const AdrModule = () => {
  const [expanded, setExpanded] = useState("ADR-001");
  return (
    <div>
      <Info type="tip">
        Les ADRs (Architecture Decision Records) documentent <em>pourquoi</em> chaque décision technique a été prise. Essentiels pour les nouveaux développeurs et les revues de code. Cliquez sur chaque ADR pour voir le détail.
      </Info>
      <div style={{ marginBottom:"16px", display:"flex", gap:"8px", flexWrap:"wrap" }}>
        {ADR_LIST.map(a => (
          <button key={a.id}
            onClick={() => setExpanded(expanded===a.id ? null : a.id)}
            style={{
              background: expanded===a.id ? C.gold+"20" : "transparent",
              border:`1px solid ${expanded===a.id ? C.gold+"50" : C.border}`,
              color: expanded===a.id ? C.gold : C.muted,
              borderRadius:"6px", padding:"5px 12px",
              fontSize:"11px", fontFamily:"monospace",
              cursor:"pointer", transition:"all 0.15s",
            }}>{a.id}</button>
        ))}
      </div>
      {ADR_LIST.map(adr => (
        <ADRCard key={adr.id} adr={adr}
          expanded={expanded===adr.id}
          onToggle={() => setExpanded(expanded===adr.id ? null : adr.id)} />
      ))}
    </div>
  );
};

// ─── ARCHITECTURE DIAGRAM ────────────────────────────────────
const ArchDiagram = () => {
  const layers = [
    {
      label:"📱 APPLICATION FLUTTER", color:C.blue, items:[
        { icon:"🖥️", name:"Presentation", detail:"Screens, Widgets, Riverpod Notifiers" },
        { icon:"🧩", name:"Domain", detail:"Models, Repositories (interfaces), UseCases" },
        { icon:"🔌", name:"Data", detail:"Supabase Client, Hive Cache, Services" },
      ]
    },
    {
      label:"☁️ SUPABASE BACKEND", color:C.teal, items:[
        { icon:"🗄️", name:"PostgreSQL 15", detail:"12 tables, 18 RLS policies, pg_trgm, tsvector" },
        { icon:"⚡", name:"Realtime", detail:"WebSocket → Messages, Notifications, Likes" },
        { icon:"📦", name:"Storage", detail:"avatars, posts-media, messages-media (Cloudflare CDN)" },
        { icon:"⚙️", name:"Edge Functions", detail:"moderate-content (Deno) + send-notification (FCM)" },
      ]
    },
    {
      label:"🌐 CLOUDFLARE", color:C.amber, items:[
        { icon:"🎬", name:"Stream", detail:"HLS adaptatif 360p→1080p, PoP Lagos+Dakar" },
        { icon:"🔧", name:"Workers", detail:"Quality selector par ECT, cache agressif" },
      ]
    },
    {
      label:"🔥 FIREBASE", color:C.red, items:[
        { icon:"🔔", name:"FCM v1", detail:"Push background + foreground, priorité HIGH DM" },
      ]
    },
    {
      label:"💰 MONÉTISATION", color:C.gold, items:[
        { icon:"📊", name:"AdMob + Meta AN", detail:"Native ads 1/5 posts, CPM $0.84, fill 94%" },
      ]
    },
  ];

  return (
    <div>
      <Sec title="Vue d'ensemble de l'architecture système" accent={C.gold} icon="🏛️">
        <div style={{ display:"flex", flexDirection:"column", gap:"12px" }}>
          {layers.map((layer, li) => (
            <div key={li} style={{
              background:C.card, border:`1px solid ${layer.color}25`,
              borderRadius:"12px", overflow:"hidden",
            }}>
              <div style={{
                background:`linear-gradient(90deg,${layer.color}15,transparent)`,
                padding:"10px 18px", borderBottom:`1px solid ${layer.color}20`,
                fontSize:"11px", fontWeight:700, color:layer.color,
                letterSpacing:"1.5px", textTransform:"uppercase",
              }}>{layer.label}</div>
              <div style={{
                display:"grid",
                gridTemplateColumns:`repeat(${Math.min(layer.items.length,4)},1fr)`,
                gap:"1px", background:C.border,
              }}>
                {layer.items.map((item, ii) => (
                  <div key={ii} style={{
                    background:C.card, padding:"14px 16px",
                  }}>
                    <div style={{ fontSize:"18px", marginBottom:"6px" }}>{item.icon}</div>
                    <div style={{ fontWeight:700, fontSize:"12.5px", color:C.text, marginBottom:"3px" }}>
                      {item.name}
                    </div>
                    <div style={{ fontSize:"11px", color:C.muted, lineHeight:"1.5" }}>
                      {item.detail}
                    </div>
                  </div>
                ))}
              </div>
            </div>
          ))}
        </div>

        {/* Flow arrows */}
        <div style={{
          background:C.dim, border:`1px solid ${C.border}`,
          borderRadius:"10px", padding:"16px 18px", marginTop:"16px",
          fontFamily:"monospace", fontSize:"11.5px", color:C.muted, lineHeight:"2",
        }}>
          <span style={{ color:C.gold, fontWeight:700 }}>Flux de données :</span>{" "}
          <span style={{ color:C.blue }}>Flutter</span>
          {" "}⟶{" "}
          <span style={{ color:C.teal }}>Supabase RPC / REST</span>
          {" "}⟶{" "}
          <span style={{ color:C.green }}>PostgreSQL</span>
          {"  |  "}
          <span style={{ color:C.blue }}>Flutter</span>
          {" "}⟵{" "}
          <span style={{ color:C.teal }}>WebSocket Realtime</span>
          {" "}⟵{" "}
          <span style={{ color:C.green }}>DB Trigger</span>
          {"  |  "}
          <span style={{ color:C.blue }}>Flutter</span>
          {" "}⟵{" "}
          <span style={{ color:C.red }}>FCM Push</span>
          {" "}⟵{" "}
          <span style={{ color:C.teal }}>Edge Function</span>
          {" "}⟵{" "}
          <span style={{ color:C.green }}>DB Trigger</span>
        </div>
      </Sec>

      <Sec title="Carte des dossiers — lib/ complète" accent={C.violet} icon="🗺️">
        <Code lang="tree — lib/" code={`lib/
├── core/
│   ├── config/
│   │   ├── env.dart            ← variables d'env (envied, obfusquées)
│   │   ├── router.dart         ← GoRouter (ShellRoute + routes nommées)
│   │   └── theme.dart          ← AppTheme.dark + couleurs + typographie
│   ├── network/
│   │   ├── supabase_client.dart ← singleton Supabase initialisé
│   │   └── dio_client.dart      ← HTTP pour Cloudflare + APIs externes
│   ├── cache/
│   │   ├── hive_service.dart    ← cache local posts, préférences
│   │   └── image_cache.dart     ← CachedNetworkImage config globale
│   └── utils/
│       ├── formatters.dart      ← timeAgo(), formatCount() (k, M)
│       └── validators.dart      ← username, caption, email
│
├── features/
│   ├── auth/
│   │   ├── screens/login_screen.dart
│   │   ├── screens/register_screen.dart
│   │   ├── providers/auth_provider.dart
│   │   └── repositories/auth_repository.dart
│   │
│   ├── feed/
│   │   ├── screens/feed_screen.dart
│   │   ├── widgets/post_card.dart
│   │   ├── widgets/video_post_card.dart
│   │   ├── widgets/feed_skeleton.dart
│   │   ├── providers/feed_provider.dart    ← @riverpod FeedNotifier
│   │   └── repositories/feed_repository.dart ← rpc('get_personalized_feed')
│   │
│   ├── post/
│   │   ├── screens/create_post_screen.dart
│   │   ├── services/upload_service.dart    ← compression + retry TUS
│   │   ├── services/cloudflare_stream_service.dart
│   │   └── models/post_model.dart          ← @freezed
│   │
│   ├── profile/
│   │   ├── screens/profile_screen.dart
│   │   ├── providers/follow_provider.dart  ← optimistic update
│   │   └── repositories/profile_repository.dart
│   │
│   ├── messaging/
│   │   ├── screens/messages_list_screen.dart
│   │   ├── screens/conversation_screen.dart
│   │   ├── widgets/message_bubble.dart
│   │   └── services/realtime_message_service.dart  ← WebSocket
│   │
│   ├── search/
│   │   ├── screens/explore_screen.dart
│   │   ├── providers/search_provider.dart  ← debounce 300ms
│   │   └── repositories/search_repository.dart ← rpc('unified_search')
│   │
│   ├── notifications/
│   │   ├── screens/notifications_screen.dart
│   │   └── services/fcm_service.dart       ← handlers bg/fg/closed
│   │
│   └── ads/
│       ├── services/ad_manager.dart         ← préchargement + fallback
│       ├── services/analytics_service.dart  ← CPM/CPC logging
│       └── widgets/native_ad_card.dart
│
├── shared/
│   ├── widgets/
│   │   ├── error_widget.dart
│   │   ├── loading_skeleton.dart
│   │   └── hashtag_text_field.dart
│   ├── models/
│   │   ├── user_model.dart
│   │   └── message_model.dart
│   └── providers/
│       ├── current_user_provider.dart
│       └── supabase_client_provider.dart
│
└── main.dart`} />
      </Sec>
    </div>
  );
};

// ═══════════════════════════════════════════════════════════════
// MAIN APP
// ═══════════════════════════════════════════════════════════════
const MAIN_TABS = [
  { id:"checklist", icon:"🚀", label:"Checklist Lancement",    accent:C.gold },
  { id:"readme",    icon:"📄", label:"README & Docs",          accent:C.violet },
  { id:"adr",       icon:"📐", label:"ADR (7 décisions)",      accent:C.amber },
  { id:"arch",      icon:"🏛️",  label:"Architecture Complète",  accent:C.teal },
  { id:"setup",     icon:"⚙️", label:"Guide Installation",     accent:C.green },
];

export default function FinalDocs() {
  const [active, setActive] = useState("checklist");
  const [clock, setClock]   = useState(new Date());

  useEffect(() => {
    const t = setInterval(() => setClock(new Date()), 1000);
    return () => clearInterval(t);
  }, []);

  const pages = {
    checklist: <LaunchChecklist />,
    readme:    <ReadmeModule />,
    adr:       <AdrModule />,
    arch:      <ArchDiagram />,
    setup:     <SetupModule />,
  };

  return (
    <div style={{ background:C.bg, minHeight:"100vh", color:C.text,
      fontFamily:"'DM Sans','Segoe UI',sans-serif" }}>

      {/* Header */}
      <div style={{
        background:"linear-gradient(135deg,#100e1a,#07060a)",
        borderBottom:`1px solid ${C.border}`,
        padding:"0 28px", height:"60px",
        display:"flex", alignItems:"center", justifyContent:"space-between",
        position:"sticky", top:0, zIndex:100,
      }}>
        <div style={{ display:"flex", alignItems:"center", gap:"14px" }}>
          <div style={{
            background:`linear-gradient(135deg,${C.gold},${C.amber})`,
            borderRadius:"10px", width:"36px", height:"36px",
            display:"flex", alignItems:"center", justifyContent:"center",
            fontSize:"18px", flexShrink:0, boxShadow:`0 0 16px ${C.gold}40`,
          }}>📘</div>
          <div>
            <div style={{ fontWeight:900, fontSize:"16px", letterSpacing:"-0.5px" }}>
              SocialApp <span style={{ color:C.gold }}>— Version Finale</span>
            </div>
            <div style={{ fontSize:"10px", color:C.muted, letterSpacing:"0.5px" }}>
              FINDEV CONSEIL NIGER · Documentation Complète · Production-Ready
            </div>
          </div>
        </div>

        <div style={{ display:"flex", alignItems:"center", gap:"12px" }}>
          <div style={{ display:"flex", gap:"6px", flexWrap:"wrap" }}>
            <Chip text="v1.0.0" color={C.green} />
            <Chip text="66 tests" color={C.blue} />
            <Chip text="7 ADRs" color={C.gold} />
            <Chip text="86.3% cov" color={C.teal} />
          </div>
          <div style={{ fontFamily:"monospace", fontSize:"11px", color:C.muted }}>
            {clock.toLocaleTimeString("fr-FR")}
          </div>
        </div>
      </div>

      {/* Tabs */}
      <div style={{
        display:"flex", overflowX:"auto", padding:"0 24px",
        borderBottom:`1px solid ${C.border}`,
        background:C.surface, gap:"2px", scrollbarWidth:"none",
      }}>
        {MAIN_TABS.map(t => (
          <button key={t.id} onClick={() => setActive(t.id)} style={{
            background:"transparent", border:"none",
            borderBottom: active===t.id ? `2px solid ${t.accent}` : "2px solid transparent",
            color: active===t.id ? t.accent : C.muted,
            padding:"15px 18px", cursor:"pointer",
            fontSize:"13px", fontWeight: active===t.id ? 800 : 400,
            whiteSpace:"nowrap", transition:"all 0.15s", fontFamily:"inherit",
          }}>
            {t.icon} {t.label}
          </button>
        ))}
      </div>

      {/* Content */}
      <div style={{ padding:"28px", maxWidth:"1060px", margin:"0 auto" }}>
        {pages[active]}
      </div>

      {/* Footer */}
      <div style={{
        borderTop:`1px solid ${C.border}`, padding:"14px 28px",
        display:"flex", justifyContent:"space-between", alignItems:"center",
        flexWrap:"wrap", gap:"10px", background:"#030208",
      }}>
        <div style={{ fontSize:"12px", color:C.muted }}>
          © 2025 <span style={{ color:C.gold }}>FINDEV CONSEIL NIGER</span> · Lawali Ibrahim Mahaman Sanoussi · Maradi, Niger
        </div>
        <div style={{ display:"flex", gap:"8px" }}>
          {["Flutter","Supabase","Cloudflare","FCM","AdMob","GitHub Actions","Fastlane"].map(t => (
            <Chip key={t} text={t} color={C.muted} />
          ))}
        </div>
      </div>
    </div>
  );
}
