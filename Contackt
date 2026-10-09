import os, textwrap, zipfile, json
root = "/mnt/data/Connectly_Starter" os.makedirs(root, exist_ok=True)
files = { "README_BN.md": r"""
Connectly — Android + Web Admin Starter
এটি একটি starter project, production-ready সম্পূর্ণ calling/live-streaming service নয়। এতে Android Kotlin app-এর starter UI/data layer, Firebase config placeholders, Firestore rules, Cloud Functions skeleton এবং React admin panel রয়েছে।
অন্তর্ভুক্ত
Android: Kotlin + Jetpack Compose, Firebase Auth/Firestore/Storage dependencies
Web admin: React + Vite + Firebase Auth/Firestore
Firestore security rules: signed-in users, conversation membership, admin custom claim
Cloud Functions: admin claim দেওয়ার trusted server-side template
Chat, group chat, file share-এর starter screen/flow
Audio/video calling ও live streaming-এর জন্য LiveKit integration points (নিজস্ব server URL/token endpoint লাগবে)
প্রথমে যা করতে হবে
Android Studio-তে android/ ফোল্ডার খুলুন।
Firebase Console-এ একটি project তৈরি করুন; Android app ID com.connectly.app যোগ করুন।
Firebase Console থেকে google-services.json ডাউনলোড করে android/app/google-services.json-এ রাখুন।
Firebase Authentication-এ Email/Password চালু করুন।
Firestore Database ও Storage চালু করুন।
firebase/ ফোল্ডারের rules deploy করুন। Storage rules-এর sample পড়ে নিজের access policy ঠিক করুন।
Web admin-এর জন্য web-admin/.env.example কপি করে .env বানিয়ে Firebase web config বসান।
web-admin/-এ npm install তারপর npm run dev চালান।
কাউকে admin করতে Firebase Admin SDK দিয়ে trusted environment থেকে custom claim admin: true দিন। Client app থেকে admin claim কখনো দেবেন না।
গুরুত্বপূর্ণ
google-services.json ও .env-এ নিজের Firebase project config বসাতে হবে; ZIP-এ secret/config নেই।
Admin panel-এর route guard UI convenience মাত্র; Firestore rules ও backend claims-ই আসল security boundary।
Video/audio call-এর জন্য LiveKit/Agora project, token backend এবং Android SDK integration সম্পন্ন করতে হবে। এখানে UI ও integration placeholders আছে, production call transport নয়।
Live stream playback/broadcast এবং push notifications-এর জন্য FCM token registration, server-side notifications, moderation/reporting, rate limits, and testing যোগ করতে হবে।
Firebase Storage billing/plan requirements may vary. Test before launch. """, "firebase/firestore.rules": r""" rules_version = '2'; service cloud.firestore { match /databases/{database}/documents { function signedIn() { return request.auth != null; } function isAdmin() { return signedIn() && request.auth.token.admin == true; } function isSelf(uid) { return signedIn() && request.auth.uid == uid; }
match /users/{uid} { allow read: if signedIn(); allow create: if isSelf(uid); allow update: if isSelf(uid) || isAdmin(); allow delete: if isAdmin(); }
match /conversations/{conversationId} { function isMember() { return signedIn() && request.auth.uid in resource.data.memberIds; } allow create: if signedIn() && request.resource.data.memberIds is list && request.auth.uid in request.resource.data.memberIds; allow read: if isMember() || isAdmin(); allow update: if isMember() || isAdmin(); allow delete: if isAdmin();
match /messages/{messageId} {
  function parentMember() {
    return signedIn() &&
      request.auth.uid in get(/databases/$(database)/documents/conversations/$(conversationId)).data.memberIds;
  }
  allow read: if parentMember() || isAdmin();
  allow create: if parentMember()
    && request.resource.data.senderId == request.auth.uid
    && request.resource.data.text is string
    && request.resource.data.text.size() <= 10000;
  allow update: if parentMember()
    && resource.data.senderId == request.auth.uid;
  allow delete: if parentMember() && resource.data.senderId == request.auth.uid || isAdmin();
}
}
match /groups/{groupId} { allow read: if signedIn() && (request.auth.uid in resource.data.memberIds || isAdmin()); allow create: if signedIn() && request.auth.uid in request.resource.data.memberIds; allow update: if signedIn() && (request.auth.uid in resource.data.memberIds || isAdmin()); allow delete: if isAdmin(); }
match /reports/{reportId} { allow create: if signedIn() && request.resource.data.reporterId == request.auth.uid; allow read, update, delete: if isAdmin(); }
match /liveSessions/{sessionId} { allow read: if signedIn(); allow create: if signedIn() && request.resource.data.hostId == request.auth.uid; allow update: if signedIn() && (resource.data.hostId == request.auth.uid || isAdmin()); allow delete: if isAdmin() || (signedIn() && resource.data.hostId == request.auth.uid); }
match /{document=**} { allow read, write: if false; } } } """, "firebase/storage.rules": r""" rules_version = '2'; service firebase.storage { match /b/{bucket}/o { function signedIn() { return request.auth != null; } function isAdmin() { return signedIn() && request.auth.token.admin == true; }
match /users/{uid}/{allPaths=**} { allow read: if signedIn(); allow write: if signedIn() && request.auth.uid == uid && request.resource.size < 20 * 1024 * 1024; } match /chat/{conversationId}/{fileName} { // For production, verify conversation membership with a backend or move to controlled uploads. allow read, write: if signedIn() || isAdmin(); } } } """, "firebase/firebase.json": json.dumps({"firestore":{"rules":"firestore.rules"},"storage":{"rules":"storage.rules"},"functions":{"source":"functions"}}, indent=2), "firebase/functions/package.json": json.dumps({"name":"connectly-functions","private":True,"engines":{"node":"20"},"main":"index.js","dependencies":{"firebase-admin":"^12.7.0","firebase-functions":"^6.1.0"}}, indent=2), "firebase/functions/index.js": r""" const { onCall, HttpsError } = require("firebase-functions/v2/https"); const { initializeApp } = require("firebase-admin/app"); const { getAuth } = require("firebase-admin/auth");
initializeApp();
// Deploy this callable only for an already-authenticated, trusted operator. // Add your own operator allowlist check before production deployment. exports.setAdminClaim = onCall(async (request) => { if (!request.auth) throw new HttpsError("unauthenticated", "Sign in required."); if (request.auth.token.admin !== true) { throw new HttpsError("permission-denied", "Existing admin claim required."); } const uid = request.data?.uid; const enabled = request.data?.enabled === true; if (typeof uid !== "string" || !uid) { throw new HttpsError("invalid-argument", "A target uid is required."); } await getAuth().setCustomUserClaims(uid, { admin: enabled }); return { ok: true, uid, admin: enabled }; }); """, "android/settings.gradle.kts": r""" pluginManagement { repositories { google(); mavenCentral(); gradlePluginPortal() } } dependencyResolutionManagement { repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS) repositories { google(); mavenCentral() } } rootProject.name = "Connectly" include(":app") """, "android/build.gradle.kts": r""" plugins { id("com.android.application") version "8.7.3" apply false id("org.jetbrains.kotlin.android") version "2.0.21" apply false id("org.jetbrains.kotlin.plugin.compose") version "2.0.21" apply false id("com.google.gms.google-services") version "4.4.2" apply false } """, "android/gradle.properties": "org.gradle.jvmargs=-Xmx2048m -Dfile.encoding=UTF-8\nandroid.useAndroidX=true\nkotlin.code.style=official\n", "android/app/build.gradle.kts": r""" plugins { id("com.android.application") id("org.jetbrains.kotlin.android") id("org.jetbrains.kotlin.plugin.compose") id("com.google.gms.google-services") } android { namespace = "com.connectly.app" compileSdk = 35 defaultConfig { applicationId = "com.connectly.app" minSdk = 24 targetSdk = 35 versionCode = 1 versionName = "1.0" } buildFeatures { compose = true } } dependencies { implementation(platform("androidx.compose:compose-bom:2024.12.01")) implementation("androidx.activity:activity-compose:1.10.0") implementation("androidx.compose.ui:ui") implementation("androidx.compose.ui:ui-tooling-preview") implementation("androidx.compose.material3:material3") implementation("androidx.navigation:navigation-compose:2.8.5") implementation(platform("com.google.firebase:firebase-bom:33.7.0")) implementation("com.google.firebase:firebase-auth") implementation("com.google.firebase:firebase-firestore") implementation("com.google.firebase:firebase-storage") implementation("com.google.firebase:firebase-messaging") debugImplementation("androidx.compose.ui:ui-tooling") } """, "android/app/src/main/AndroidManifest.xml": r"""               """, "android/app/src/main/java/com/connectly/app/MainActivity.kt": r""" package com.connectly.app
import android.os.Bundle import androidx.activity.ComponentActivity import androidx.activity.compose.setContent import androidx.compose.foundation.layout.* import androidx.compose.foundation.lazy.LazyColumn import androidx.compose.foundation.lazy.items import androidx.compose.material3.* import androidx.compose.runtime.* import androidx.compose.ui.Modifier import androidx.compose.ui.unit.dp import com.google.firebase.FirebaseApp import com.google.firebase.auth.FirebaseAuth import com.google.firebase.firestore.FirebaseFirestore import com.google.firebase.firestore.Query
class MainActivity : ComponentActivity() { override fun onCreate(savedInstanceState: Bundle?) { super.onCreate(savedInstanceState) FirebaseApp.initializeApp(this) setContent { MaterialTheme { ConnectlyApp() } } } }
@Composable fun ConnectlyApp() { var email by remember { mutableStateOf("") } var password by remember { mutableStateOf("") } var status by remember { mutableStateOf("Welcome to Connectly") } var signedIn by remember { mutableStateOf(FirebaseAuth.getInstance().currentUser != null) } var message by remember { mutableStateOf("") } var messages by remember { mutableStateOf(listOf()) }
LaunchedEffect(signedIn) {
    if (signedIn) {
        FirebaseFirestore.getInstance().collection("publicDemoMessages")
            .orderBy("createdAt", Query.Direction.ASCENDING).limit(50)
            .addSnapshotListener { snap, err ->
                if (err != null) { status = err.localizedMessage ?: "Could not load messages"; return@addSnapshotListener }
                messages = snap?.documents?.map { "${it.getString("senderName") ?: "User"}: ${it.getString("text") ?: ""}" } ?: emptyList()
            }
    }
}

Scaffold { padding ->
    LazyColumn(
        modifier = Modifier.fillMaxSize().padding(padding).padding(20.dp),
        verticalArrangement = Arrangement.spacedBy(12.dp)
    ) {
        item { Text("Connectly", style = MaterialTheme.typography.headlineLarge) }
        item { Text("Chat • Calls • Live", style = MaterialTheme.typography.titleMedium) }
        item { Text(status) }

        if (!signedIn) {
            item { OutlinedTextField(email, { email = it }, label = { Text("Email") }, modifier = Modifier.fillMaxWidth()) }
            item { OutlinedTextField(password, { password = it }, label = { Text("Password") }, modifier = Modifier.fillMaxWidth()) }
            item {
                Row(horizontalArrangement = Arrangement.spacedBy(8.dp)) {
                    Button(onClick = {
                        FirebaseAuth.getInstance().signInWithEmailAndPassword(email.trim(), password)
                            .addOnSuccessListener { signedIn = true; status = "Signed in" }
                            .addOnFailureListener { status = it.localizedMessage ?: "Sign in failed" }
                    }) { Text("Login") }
                    OutlinedButton(onClick = {
                        FirebaseAuth.getInstance().createUserWithEmailAndPassword(email.trim(), password)
                            .addOnSuccessListener { signedIn = true; status = "Account created" }
                            .addOnFailureListener { status = it.localizedMessage ?: "Sign up failed" }
                    }) { Text("Sign up") }
                }
            }
        } else {
            item {
                Row(horizontalArrangement = Arrangement.spacedBy(8.dp)) {
                    OutlinedButton(onClick = {
                        FirebaseAuth.getInstance().signOut()
                        signedIn = false
                        status = "Signed out"
                    }) { Text("Sign out") }
                    OutlinedButton(onClick = { status = "Call integration: configure LiveKit/Agora token service first." }) { Text("Audio / Video") }
                    OutlinedButton(onClick = { status = "Live streaming requires a configured LiveKit/Agora project." }) { Text("Go Live") }
                }
            }
            item { Text("Demo public chat", style = MaterialTheme.typography.titleLarge) }
            items(messages) { Text(it, modifier = Modifier.fillMaxWidth().padding(vertical = 4.dp)) }
            item {
                OutlinedTextField(message, { message = it }, label = { Text("Message") }, modifier = Modifier.fillMaxWidth())
            }
            item {
                Button(onClick = {
                    val user = FirebaseAuth.getInstance().currentUser
                    if (user == null || message.isBlank()) { status = "Sign in and enter a message"; return@Button }
                    FirebaseFirestore.getInstance().collection("publicDemoMessages").add(
                        mapOf("senderId" to user.uid, "senderName" to (user.email ?: "User"),
                            "text" to message.trim(), "createdAt" to com.google.firebase.firestore.FieldValue.serverTimestamp())
                    ).addOnSuccessListener { message = "" }
                        .addOnFailureListener { status = it.localizedMessage ?: "Send failed" }
                }) { Text("Send message") }
            }
        }
        item { HorizontalDivider() }
        item { Text("Starter modules", style = MaterialTheme.typography.titleMedium) }
        item { Text("Private chat, groups, file uploads, push notifications, moderation and call/live SDKs are integration steps documented in README_BN.md.") }
    }
}
} """, "android/app/google-services.json.example": json.dumps({"project_info":{"project_number":"REPLACE_ME","project_id":"REPLACE_ME","storage_bucket":"REPLACE_ME.appspot.com"},"client":[{"client_info":{"mobilesdk_app_id":"REPLACE_ME","android_client_info":{"package_name":"com.connectly.app"}},"api_key":[{"current_key":"REPLACE_ME"}],"services":{}}],"configuration_version":"1"}, indent=2), "web-admin/package.json": json.dumps({"name":"connectly-admin","version":"1.0.0","private":True,"type":"module","scripts":{"dev":"vite --host 0.0.0.0","build":"vite"},"dependencies":{"firebase":"^11.1.0","react":"^18.3.1","react-dom":"^18.3.1"},"devDependencies":{"@vitejs/plugin-react":"^4.3.4","vite":"^6.0.5"}}, indent=2), "web-admin/index.html": 'Connectly Admin', "web-admin/vite.config.js": "import { defineConfig } from 'vite'; import react from '@vitejs/plugin-react'; export default defineConfig({plugins:[react()]});", "web-admin/.env.example": """VITE_FIREBASE_API_KEY=replace_me VITE_FIREBASE_AUTH_DOMAIN=replace_me VITE_FIREBASE_PROJECT_ID=replace_me VITE_FIREBASE_STORAGE_BUCKET=replace_me VITE_FIREBASE_MESSAGING_SENDER_ID=replace_me VITE_FIREBASE_APP_ID=replace_me """, "web-admin/src/firebase.js": r""" import { initializeApp } from 'firebase/app'; import { getAuth } from 'firebase/auth'; import { getFirestore } from 'firebase/firestore';
const config = { apiKey: import.meta.env.VITE_FIREBASE_API_KEY, authDomain: import.meta.env.VITE_FIREBASE_AUTH_DOMAIN, projectId: import.meta.env.VITE_FIREBASE_PROJECT_ID, storageBucket: import.meta.env.VITE_FIREBASE_STORAGE_BUCKET, messagingSenderId: import.meta.env.VITE_FIREBASE_MESSAGING_SENDER_ID, appId: import.meta.env.VITE_FIREBASE_APP_ID, }; const app = initializeApp(config); export const auth = getAuth(app); export const db = getFirestore(app); """, "web-admin/src/main.jsx": r""" import React, { useEffect, useState } from 'react'; import { createRoot } from 'react-dom/client'; import { onAuthStateChanged, signInWithEmailAndPassword, signOut } from 'firebase/auth'; import { collection, getDocs, limit, query, orderBy } from 'firebase/firestore'; import { auth, db } from './firebase'; import './style.css';
function App() { const [user, setUser] = useState(null); const [email, setEmail] = useState(''); const [password, setPassword] = useState(''); const [error, setError] = useState(''); const [users, setUsers] = useState([]); const [reports, setReports] = useState([]); const [tab, setTab] = useState('overview'); const [admin, setAdmin] = useState(false);
useEffect(() => onAuthStateChanged(auth, async (u) => { setUser(u); if (u) { const token = await u.getIdTokenResult(true); setAdmin(token.claims.admin === true); } else setAdmin(false); }), []);
async function loadData() { try { const [u, r] = await Promise.all([ getDocs(query(collection(db, 'users'), limit(100))), getDocs(query(collection(db, 'reports'), limit(100))) ]); setUsers(u.docs.map(d => ({ id: d.id, ...d.data() }))); setReports(r.docs.map(d => ({ id: d.id, ...d.data() }))); setError(''); } catch (e) { setError(e.message); } } useEffect(() => { if (user && admin) loadData(); }, [user, admin]);
if (!user) return ConnectlyAdmin sign inUse an account granted the admin custom claim.{error && {error}}<form onSubmit={async e => { e.preventDefault(); setError(''); try { await signInWithEmailAndPassword(auth, email, password); } catch (x) { setError(x.message); } }}>Email<input type="email" value={email} onChange={e=>setEmail(e.target.value)} required />Password<input type="password" value={password} onChange={e=>setPassword(e.target.value)} required />Sign in;
if (!admin) return ConnectlyAccess deniedThis account does not have the Firebase admin custom claim.<button onClick={()=>signOut(auth)}>Sign out;
return ConnectlyADMIN{['overview','users','reports','live sessions'].map(x=><button className={tab===x?'nav active':'nav'} key={x} onClick={()=>setTab(x)}>{x})}<button className="nav logout" onClick={()=>signOut(auth)}>Sign outWORKSPACE / {tab.toUpperCase()}{tab==='overview'?'Dashboard':tab[0].toUpperCase()+tab.slice(1)}Refresh data{error&&{error}}{tab==='overview'?<>Loaded users{users.length}Open reports{reports.length}Signed-in adminVerifiedAdmin quick notes</
