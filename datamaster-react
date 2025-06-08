import React, { useState, useEffect, useRef, useCallback } from 'react';
import { initializeApp } from 'firebase/app';
import { getAuth, signInAnonymously, signInWithCustomToken, onAuthStateChanged } from 'firebase/auth';
import { getFirestore, doc, getDoc, setDoc, collection, query, where, getDocs, serverTimestamp } from 'firebase/firestore';

// Define Firebase Config from global __firebase_config (provided by Canvas environment)
// Fallback to a default structure if not available, though in Canvas it should be.
const firebaseConfig = typeof __firebase_config !== 'undefined' 
    ? JSON.parse(__firebase_config) 
    : {
        apiKey: "YOUR_API_KEY", // Replace with your actual key if running outside Canvas
        authDomain: "YOUR_AUTH_DOMAIN",
        projectId: "YOUR_PROJECT_ID",
        storageBucket: "YOUR_STORAGE_BUCKET",
        messagingSenderId: "YOUR_MESSAGING_SENDER_ID",
        appId: "YOUR_APP_ID",
        measurementId: "YOUR_MEASUREMENT_ID"
    };

// All read/write operations will now use this single APP_ID.
const APP_ID = firebaseConfig.projectId;

// Define feeds outside of the component to prevent re-creation on re-renders
const feeds = {
    'datamaster': {
        id: 'datamaster',
        uri: 'at://did:plc:lptjvw6ut224kwrj7ub3sqbe/app.bsky.feed.generator/datamaster',
        displayName: 'Home',
        versions: [{ name: 'Default', src: 'https://graze.social/feeds/embed/8796' }],
        icon: 'home',
        blueskyProfileUrl: 'https://bsky.app/profile/did:plc:lptjvw6ut224kwrj7ub3sqbe/feed/datamaster',
        titleIconScale: '1',
        analyzeData: false
    },
    'urbanismPlus': {
        id: 'urbanismPlus',
        uri: 'at://did:plc:lptjvw6ut224kwrj7ub3sqbe/app.bsky.feed.generator/aaaotfjzjplna',
        displayName: 'Urbanism+',
        versions: [{ name: 'Default', src: 'https://graze.social/feeds/embed/3654' }],
        icon: 'https://raw.githubusercontent.com/femavibes/datamaster/refs/heads/main/uplus.jpg',
        blueskyProfileUrl: 'https://bsky.app/profile/did:plc:lptjvw6ut224kwrj7ub3sqbe/feed/aaaotfjzjplna',
        titleIconScale: '1.0',
        analyzeData: true
    },
    'bandcamp': {
        id: 'bandcamp',
        uri: 'at://did:plc:lptjvw6ut224kwrj7ub3sqbe/app.bsky.feed.generator/bandcamp',
        displayName: 'Bandcamp',
        versions: [
            { name: 'Trending', src: 'https://graze.social/feeds/embed/8723' },
            { name: 'New', src: 'https://graze.social/feeds/embed/8768' }
        ],
        icon: 'https://raw.githubusercontent.com/femavibes/datamaster/refs/heads/main/bandcamp7796.logowik.com.webp',
        blueskyProfileUrl: 'https://bsky.app/profile/did:plc:lptjvw6ut224kwrj7ub3sqbe/feed/bandcamp',
        titleIconScale: '1.3225',
        analyzeData: true
    },
    'transitSky': {
        id: 'transitSky',
        uri: 'at://did:plc:lptjvw6ut224kwrj7ub3sqbe/app.bsky.feed.generator/aaaic34mdicfg',
        displayName: 'TransitSky',
        versions: [{ name: 'Default', src: 'https://graze.social/feeds/embed/5511' }],
        icon: 'train',
        blueskyProfileUrl: 'https://bsky.app/profile/did:plc:lptjvw6ut224kwrj7ub3sqbe/feed/aaaic34mdicfg',
        titleIconScale: '1',
        analyzeData: true
    }
};

// Stop words for term analysis (still relevant for hashtags)
const stopWords = new Set([
    'a', 'an', 'the', 'and', 'or', 'but', 'for', 'nor', 'on', 'at', 'to', 'from', 'by',
    'with', 'in', 'of', 'is', 'am', 'are', 'was', 'were', 'be', 'been', 'being', 'have',
    'has', 'had', 'do', 'does', 'did', 'not', 'no', 'yes', 'it', 'its', 'he', 'she', 'they',
    'we', 'you', 'i', 'me', 'him', 'her', 'us', 'them', 'this', 'that', 'these', 'those',
    'my', 'your', 'his', 'her', 'our', 'their', 'which', 'who', 'whom', 'where', 'when',
    'why', 'how', 'what', 'then', 'than', 'as', 'so', 'if', 'else', 'about', 'just', 'can',
    'will', 'would', 'should', 'could', 'get', 'like', 'one', 'two', 'new', 'time', 'good',
    'great', 'very', 'much', 'more', 'also', 'up', 'down', 'out', 'in', 'on', 'off', 'all',
    'any', 'some', 'such', 'only', 'very', 'through', 'into', 'from', 'here', 'there', 'when',
    'why', 'how', 'all', 'any', 'both', 'each', 'few', 'more', 'most', 'other',
    'some', 'such', 'no', 'nor', 'not', 'only', 'own', 'same', 'so', 'than', 'too', 'very',
    's', 't', 'can', 'will', 'just', 'don', 'should', 'now'
]);

// Theme definitions (moved outside for performance)
const themes = {
    'none': {
        backgroundImage: 'none',
        backgroundSize: 'auto',
        backgroundRepeat: 'no-repeat',
        backgroundPosition: 'center center',
        backgroundAttachment: 'fixed'
    },
    'pride': {
        backgroundImage: 'url(https://raw.githubusercontent.com/femavibes/datamaster/refs/heads/main/flag-36423_1920.png)',
        backgroundSize: 'cover',
        backgroundRepeat: 'no-repeat',
        backgroundPosition: 'center center',
        backgroundAttachment: 'fixed'
    },
    'transgenderPride': {
        backgroundImage: 'url(https://raw.githubusercontent.com/femavibes/datamaster/refs/heads/main/transgender-flag.png)',
        backgroundSize: 'cover',
        backgroundRepeat: 'no-repeat',
        backgroundPosition: 'center center',
        backgroundAttachment: 'fixed'
    },
    'blm': {
        backgroundImage: 'url(https://raw.githubusercontent.com/femavibes/datamaster/refs/heads/main/BLM_flag.svg.png)',
        backgroundSize: '80px 80px',
        backgroundRepeat: 'repeat',
        backgroundPosition: 'center center',
        backgroundAttachment: 'fixed'
    }
};

const RECONNECT_DELAY = 10000; // 10 seconds
const DEBOUNCE_DELAY = 60000; // 1 minute (60 seconds)

// Helper function to fetch headline from a URL
async function fetchHeadline(url) {
    try {
        const response = await fetch(url);
        if (!response.ok) {
            console.warn(`Failed to fetch ${url} for headline: ${response.status} ${response.statusText}`);
            return null;
        }
        const contentType = response.headers.get('content-type');
        if (!contentType || !contentType.includes('text/html')) {
            console.warn(`Skipping headline fetch for non-HTML content at ${url}. Content-Type: ${contentType}`);
            return null;
        }
        const text = await response.text();
        const match = text.match(/<title[^>]*>([^<]+)<\/title>/i);
        return match && match[1] ? match[1].trim() : null;
    } catch (e) {
        console.warn(`CORS or network error fetching headline for ${url}:`, e.message);
        if (e instanceof TypeError && e.message.includes('Failed to fetch')) {
            console.warn(`CORS likely prevented fetching headline for ${url}. Headline will not be displayed.`);
        }
        return null;
    }
}

// Helper function to extract simplified base domain from a URL
function getSimplifiedDomain(url) {
    let hostname;
    try {
        if (!url.startsWith('http://') && !url.startsWith('https://')) {
            hostname = new URL(`http://${url}`).hostname.toLowerCase();
        } else {
            hostname = new URL(url).hostname.toLowerCase();
        }
    } catch (e) {
        console.warn(`Could not parse URL "${url}", returning as is for domain extraction attempt:`, e);
        const parts = url.toLowerCase().split('.');
        if (parts.length > 1) {
            const potentialDomain = parts[parts.length - 2] || parts[0];
            return potentialDomain.replace(/^www\./, '');
        }
        return 'Invalid Domain';
    }

    let domain = hostname.replace(/^www\./, '');

    const knownTlds = [
        '.co.uk', '.com.au', '.org.uk', '.net.au', '.ac.uk', '.gov.uk',
        '.co.nz', '.org.nz', '.net.nz', '.ac.nz',
        '.co.jp', '.ne.jp', '.or.jp', '.ac.jp', '.ad.jp', '.ed.jp', '.go.jp', '.gr.jp', '.lg.jp',
        '.com.br', '.net.br', '.org.br',
        '.com.cn', '.net.cn', '.org.cn',
        '.com.hk', '.net.hk', '.org.hk',
        '.com', '.org', '.net', '.info', '.biz', '.co', '.io', '.app', '.news', '.blog', '.tech', '.store', '.online', '.site', '.xyz'
    ].sort((a, b) => b.length - a.length);

    for (const tld of knownTlds) {
        if (domain.endsWith(tld)) {
            domain = domain.substring(0, domain.length - tld.length);
            break;
        }
    }

    const parts = domain.split('.');
    if (parts.length > 1) {
        return parts[parts.length - 1];
    } else {
        return domain;
    }
}

// Function to canonicalize domain names for display
function canonicalizeDisplayDomains(rawDomainDataWithSources) {
    const canonicalMap = {
        "streetsblog.org": "streetsblog",
        "theguardian.com": "theguardian",
        "bbc.co.uk": "bbc",
        "bbc.com": "bbc"
    };

    const finalDisplayData = {};

    for (const simplifiedDomainKey in rawDomainDataWithSources) {
        const domainEntry = rawDomainDataWithSources[simplifiedDomainKey];

        let canonicalName = simplifiedDomainKey;
        for (const mapKey in canonicalMap) {
            if (simplifiedDomainKey === mapKey) {
                canonicalName = canonicalMap[mapKey];
                break;
            }
        }

        if (!finalDisplayData[canonicalName]) {
            finalDisplayData[canonicalName] = { count: 0, original_full_domains_list: new Set() };
        }

        finalDisplayData[canonicalName].count += domainEntry.count;

        for (const sourceDomain in domainEntry.sources) {
            finalDisplayData[canonicalName].original_full_domains_list.add(sourceDomain);
        }
    }

    for (const canonicalName in finalDisplayData) {
        finalDisplayData[canonicalName].original_full_domains_list =
            Array.from(finalDisplayData[canonicalName].original_full_domains_list).sort();
    }

    return finalDisplayData;
}

// Helper function to getYYYY-MM-DD format for daily aggregate document IDs
function getYYYYMMDD(date) {
    const year = date.getFullYear();
    const month = (date.getMonth() + 1).toString().padStart(2, '0');
    const day = date.getDate().toString().padStart(2, '0');
    return `${year}-${month}-${day}`;
}

/**
 * Recursively extracts link, title, description, and the first found thumbnail URL from a Bluesky embed object.
 */
function extractEmbedData(embed) {
    let link = null;
    let title = null;
    let description = null;
    let thumbnailUrl = null;

    if (!embed) return { link, title, description, thumbnailUrl };

    const findContent = (currentEmbed) => {
        if (!currentEmbed) return;

        if (currentEmbed.$type === 'app.bsky.embed.external' && currentEmbed.external?.uri) {
            if (!link) link = currentEmbed.external.uri;
            if (!title) title = currentEmbed.external.title;
            if (!description) description = currentEmbed.external.description;
            if (!thumbnailUrl) thumbnailUrl = currentEmbed.external.thumb;
        } else if (currentEmbed.$type === 'app.bsky.embed.images' && currentEmbed.images?.length > 0) {
            if (!thumbnailUrl) thumbnailUrl = currentEmbed.images[0].thumb;
        } else if (currentEmbed.$type === 'app.bsky.embed.recordWithMedia') {
            findContent(currentEmbed.media);
            findContent(currentEmbed.record?.record?.value?.embed);
        } else if (currentEmbed.$type === 'app.bsky.embed.record' && currentEmbed.record?.value?.embed) {
            findContent(currentEmbed.record.value.embed);
        }
    };

    findContent(embed);
    return { link, title, description, thumbnailUrl };
}

function App() {
    // Firebase states
    const [firebaseApp, setFirebaseApp] = useState(null);
    const [db, setDb] = useState(null);
    const [auth, setAuth] = useState(null);
    const [currentUserId, setCurrentUserId] = useState(null);
    const [isFirebaseReady, setIsFirebaseReady] = useState(false);
    const [connectionStatus, setConnectionStatus] = useState('Initializing...');

    // App data states
    const [currentActiveFeedId, setCurrentActiveFeedId] = useState(() => localStorage.getItem('activeFeedId') || 'datamaster');
    const [currentFeedData, setCurrentFeedData] = useState({
        termCounts: {}, linkCounts: {}, domainCounts: {}, contentTypeCounts: {},
        linkCardsData: {}, imageCardsData: {}, posterCounts: {}, posterLikes: {}, mentionCounts: {}
    });
    const [resolvedDidsCache, setResolvedDidsCache] = useState({});

    // UI display states
    const [currentSelectedTimeframe, setCurrentSelectedTimeframe] = useState(() => localStorage.getItem('selectedTimeframe') || 'allTime');
    const [currentPosterMentionsView, setCurrentPosterMentionsView] = useState(() => localStorage.getItem('posterMentionsView') || 'posters');
    const [postsToDisplay, setPostsToDisplay] = useState([]); // For the live feed (currently hidden)
    const [settingsModalOpen, setSettingsModalOpen] = useState(false);
    const [linkPostsModalOpen, setLinkPostsModalOpen] = useState(false);
    const [linkPostsModalContent, setLinkPostsModalContent] = useState({ title: '', posts: [] });
    const [plusButtonModalOpen, setPlusButtonModalOpen] = useState(false);

    // Theme and Dark Mode states
    const [darkModeEnabled, setDarkModeEnabled] = useState(() => localStorage.getItem('darkModeEnabled') === 'true');
    const [selectedTheme, setSelectedTheme] = useState(() => localStorage.getItem('selectedTheme') || 'none');

    // WebSocket refs
    const wsRef = useRef(null);
    const reconnectIntervalRef = useRef(null);
    const currentConnectingFeedIdRef = useRef(null);

    // Debounce ref for Firestore updates
    const accumulatedUpdatesRef = useRef({});
    const debounceTimerRef = useRef(null);

    // Initial Firebase & Auth setup
    useEffect(() => {
        try {
            const appInstance = initializeApp(firebaseConfig);
            const dbInstance = getFirestore(appInstance);
            const authInstance = getAuth(appInstance);

            setFirebaseApp(appInstance);
            setDb(dbInstance);
            setAuth(authInstance);

            const initialAuthToken = typeof __initial_auth_token !== 'undefined' ? __initial_auth_token : null;

            onAuthStateChanged(authInstance, async (user) => {
                if (user) {
                    setCurrentUserId(user.uid);
                    setIsFirebaseReady(true);
                    console.log("Firebase authenticated and ready. User ID:", user.uid);
                } else {
                    try {
                        if (initialAuthToken) {
                            await signInWithCustomToken(authInstance, initialAuthToken);
                            console.log("Signed in with custom token.");
                        } else {
                            await signInAnonymously(authInstance);
                            console.log("Signed in anonymously.");
                        }
                    } catch (signInError) {
                        console.warn("Firebase: Attempted signInWithCustomToken failed, falling back to signInAnonymously:", signInError);
                        try {
                            await signInAnonymously(authInstance);
                        } catch (anonymousSignInError) {
                            console.error("Firebase Auth error (after fallback):", anonymousSignInError);
                            setConnectionStatus(`Auth Error: ${anonymousSignInError.message}`);
                            setIsFirebaseReady(false);
                        }
                    }
                }
            });
        } catch (error) {
            console.error("Firebase initialization failed:", error);
            setConnectionStatus(`Firebase Init Error: ${error.message}`);
        }
    }, []); // Run only once on component mount

    // Apply dark mode and theme when state changes
    useEffect(() => {
        if (darkModeEnabled) {
            document.body.classList.add('dark-mode');
        } else {
            document.body.classList.remove('dark-mode');
        }
        localStorage.setItem('darkModeEnabled', darkModeEnabled);
    }, [darkModeEnabled]);

    useEffect(() => {
        const theme = themes[selectedTheme];
        if (theme) {
            Object.keys(theme).forEach(key => {
                document.body.style[key] = theme[key];
            });
        } else {
            // Fallback to 'none' if themeName is not found
            Object.keys(themes['none']).forEach(key => {
                document.body.style[key] = themes['none'][key];
            });
        }
        localStorage.setItem('selectedTheme', selectedTheme);
    }, [selectedTheme]);

    // Firestore data fetchers (memoized with useCallback)
    const fetchResolvedDids = useCallback(async () => {
        if (!db || !isFirebaseReady || !currentUserId) {
            console.warn("Fetch Resolved DIDs: Cannot fetch: Firebase not ready or user not authenticated.");
            return;
        }
        const resolvedDidsCollectionRef = collection(db, 'artifacts', APP_ID, 'public', 'data', 'resolvedBlueskyDIDs');
        const q = query(resolvedDidsCollectionRef);
        try {
            const querySnapshot = await getDocs(q);
            const fetchedDids = {};
            querySnapshot.forEach(docSnap => {
                fetchedDids[docSnap.id] = docSnap.data();
            });
            setResolvedDidsCache(fetchedDids);
            console.log("Fetch Resolved DIDs: Fetched resolved DIDs from Firestore.");
        } catch (error) {
            console.error("Fetch Resolved DIDs: Error fetching resolved DIDs:", error);
        }
    }, [db, isFirebaseReady, currentUserId]);

    const fetchFeedSummaryData = useCallback(async (feedId) => {
        if (!db || !isFirebaseReady || !currentUserId) {
            console.warn("Firestore Fetcher: Cannot fetch summary: Firebase not ready or user not authenticated.");
            return;
        }

        if (!feeds[feedId].analyzeData) {
            console.log(`Firestore Fetcher: Skipping data fetch for ${feedId} (analyzeData is false).`);
            setCurrentFeedData({
                termCounts: {}, linkCounts: {}, domainCounts: {}, contentTypeCounts: {},
                linkCardsData: {}, imageCardsData: {}, posterCounts: {}, posterLikes: {}, mentionCounts: {}
            });
            return;
        }

        const analysisDocRef = doc(db, 'artifacts', APP_ID, 'public', 'data', 'feeds', feedId, 'summaries', 'allTime');
        try {
            const docSnap = await getDoc(analysisDocRef);
            if (docSnap.exists()) {
                const data = docSnap.data();
                // Ensure thumbnailUrl is initialized for existing linkCardsData
                const updatedLinkCardsData = data.linkCardsData || {};
                for (const url in updatedLinkCardsData) {
                    if (updatedLinkCardsData[url].thumbnailUrl === undefined) {
                        updatedLinkCardsData[url].thumbnailUrl = null;
                    }
                }

                setCurrentFeedData({
                    termCounts: data.termCounts || {},
                    linkCounts: data.linkCounts || {},
                    domainCounts: data.domainCounts || {},
                    contentTypeCounts: data.contentTypeCounts || {},
                    linkCardsData: updatedLinkCardsData,
                    imageCardsData: data.imageCardsData || {},
                    posterCounts: data.posterCounts || {},
                    posterLikes: data.posterLikes || {},
                    mentionCounts: data.mentionCounts || {}
                });
                console.log(`Firestore Fetcher: All-Time data for ${feedId} received.`);
            } else {
                console.warn(`Firestore Fetcher: All-Time summary document for ${feedId} does not exist. Initializing with empty counts locally.`);
                setCurrentFeedData({
                    termCounts: {}, linkCounts: {}, domainCounts: {}, contentTypeCounts: {},
                    linkCardsData: {}, imageCardsData: {}, posterCounts: {}, posterLikes: {}, mentionCounts: {}
                });
            }
        } catch (error) {
            console.error(`Firestore Fetcher: Error fetching all-time summary for ${feedId}:`, error);
            setConnectionStatus(`Firestore Fetch Error: ${error.message}`);
        }
    }, [db, isFirebaseReady, currentUserId]);

    const resolveBlueskyDidAndProfile = useCallback(async (did) => {
        if (!did || !did.startsWith('did:plc:')) {
            console.warn(`DID Resolution: Attempted to resolve unsupported DID type or invalid DID: ${did}`);
            return null;
        }

        if (resolvedDidsCache[did]) {
            return resolvedDidsCache[did];
        }

        let resolvedHandle = '';
        let resolvedDisplayName = '';
        let resolvedAvatar = null;

        try {
            const profileApiUrl = `https://public.api.bsky.app/xrpc/app.bsky.actor.getProfile?actor=${did}`;
            const profileResponse = await fetch(profileApiUrl);

            if (!profileResponse.ok) {
                const errorText = await profileResponse.text();
                console.warn(`DID Resolution: Failed to fetch profile from Bluesky AppView for ${did}: HTTP status ${profileResponse.status} - ${errorText}`);
                resolvedHandle = `[Error Fetching Handle]`;
                resolvedDisplayName = 'Display name not available';
            } else {
                const profileData = await profileResponse.json();
                if (profileData) {
                    resolvedHandle = profileData.handle || `[No Handle]`;
                    resolvedDisplayName = profileData.displayName || 'Display name not set';
                    resolvedAvatar = profileData.avatar || null;
                }
            }

            if (db && currentUserId) {
                const resolvedDidsCollectionRef = collection(db, 'artifacts', APP_ID, 'public', 'data', 'resolvedBlueskyDIDs');
                await setDoc(doc(resolvedDidsCollectionRef, did), {
                    did: did,
                    handle: resolvedHandle,
                    displayName: resolvedDisplayName,
                    avatar: resolvedAvatar,
                    timestamp: serverTimestamp()
                }, { merge: true });
                setResolvedDidsCache(prev => ({ ...prev, [did]: { handle: resolvedHandle, displayName: resolvedDisplayName, avatar: resolvedAvatar, timestamp: new Date() } }));
            }
            return { handle: resolvedHandle, displayName: resolvedDisplayName, avatar: resolvedAvatar };
        } catch (err) {
            console.error(`DID Resolution: Error resolving DID ${did} or fetching profile:`, err);
            return null;
        }
    }, [db, currentUserId, resolvedDidsCache]);

    // WebSocket Connection Logic (memoized)
    const connectWebSocket = useCallback((feedId) => {
        if (!feeds[feedId].analyzeData) {
            console.log(`[connectWebSocket] Skipping WebSocket connection for ${feedId} (analyzeData is false).`);
            stopWebSocket();
            setConnectionStatus(`No real-time data for ${feeds[feedId].displayName}`);
            return;
        }

        if (!isFirebaseReady || !feeds[feedId]) {
            console.warn(`[connectWebSocket] Firebase not ready or invalid feedId (${feedId}), delaying connection. isFirebaseReady: ${isFirebaseReady}.`);
            setTimeout(() => connectWebSocket(feedId), 1000);
            return;
        }

        if (currentConnectingFeedIdRef.current === feedId && wsRef.current && (wsRef.current.readyState === WebSocket.CONNECTING || wsRef.current.readyState === WebSocket.OPEN)) {
            console.log(`[connectWebSocket] Already in CONNECTING/OPEN state for ${feedId}. Aborting duplicate attempt.`);
            return;
        }

        stopWebSocket();

        currentConnectingFeedIdRef.current = feedId;
        const feedUri = feeds[feedId].uri;
        const websocketUrl = `wss://api.graze.social/app/contrail?feed=${encodeURIComponent(feedUri)}`;

        setConnectionStatus(`Connecting to ${feeds[feedId].displayName} feed via Contrails...`);

        const newWs = new WebSocket(websocketUrl);
        wsRef.current = newWs;

        newWs.onopen = () => {
            console.log(`[WS: ${feedId}] WebSocket connected successfully!`);
            setConnectionStatus(`Connected to ${feeds[feedId].displayName}`);
            if (reconnectIntervalRef.current) {
                clearInterval(reconnectIntervalRef.current);
                reconnectIntervalRef.current = null;
            }
            currentConnectingFeedIdRef.current = null;
        };

        newWs.onmessage = async (event) => {
            try {
                const data = JSON.parse(event.data);
                let authorDid = data.did || (data.commit?.did) || 'unknown-did';
                let postText = '';
                let timestamp = '';
                let postCid = data.cid || (data.commit?.cid);
                let postFacets = [];

                if (data.commit && data.commit.record && data.commit.record.$type === 'app.bsky.feed.post') {
                    const post = data.commit.record;
                    postText = String(post.text || '');
                    timestamp = post.createdAt || new Date().toISOString();
                    const postUri = data.commit.uri;
                    postFacets = post.facets || [];

                    // Store raw post in Firestore
                    if (db && currentUserId && postUri && postCid) {
                        const postDocRef = doc(db, 'artifacts', APP_ID, 'public', 'data', 'posts', postUri);
                        await setDoc(postDocRef, {
                            $type: post.$type, uri: postUri, cid: postCid, repo: authorDid, seq: data.seq,
                            createdAt: timestamp, text_content: postText, langs: post.langs || [],
                            facets: postFacets, embed: post.embed || null, author_did: authorDid,
                            hashtags: (postText.toLowerCase().match(/#\b\w+\b/g) || []).map(tag => tag.substring(1)),
                            links_extracted: [], images_extracted: [], quotes_extracted: []
                        }, { merge: true }).catch(e => console.error(`Firestore: Error saving raw post ${postUri}:`, e));
                    }

                    const resolvedInfo = await resolveBlueskyDidAndProfile(authorDid);
                    setPostsToDisplay(prevPosts => {
                        const newPost = {
                            content: postText,
                            displayName: resolvedInfo?.displayName || 'User',
                            handle: resolvedInfo?.handle ? `@${resolvedInfo.handle}` : authorDid,
                            timestamp: new Date(timestamp).toLocaleString()
                        };
                        const updatedPosts = [newPost, ...prevPosts];
                        return updatedPosts.slice(0, 50);
                    });

                    debouncedUpdateFirestoreAnalysis(feedId, authorDid, postText, post.embed, postUri, timestamp, postFacets);
                }
            } catch (e) {
                console.error(`[WS: ${feedId} - onmessage] Failed to parse message or process data:`, e);
            }
        };

        newWs.onerror = (error) => {
            console.error(`[WS: ${feedId}] WebSocket error:`, error);
            setConnectionStatus(`WebSocket error for ${feeds[feedId].displayName}. Attempting reconnect...`);
        };

        newWs.onclose = (event) => {
            console.warn(`[WS: ${feedId}] WebSocket closed. Code: ${event.code}, Reason: "${event.reason || 'No reason provided'}"`);
            if (wsRef.current !== newWs) {
                console.log(`[WS: ${feedId}] Closed instance is not the current global active WS. Ignoring.`);
                return;
            }
            wsRef.current = null;

            if (event.code === 1000 && event.reason === "Stopping old connection") {
                console.log(`[WS: ${feedId}] Cleanly stopped by stopWebSocket function. No reconnection needed.`);
                return;
            }

            let disconnectMessage = `Disconnected from ${feeds[feedId].displayName}. Code: ${event.code}`;
            if (event.reason) {
                disconnectMessage += `, Reason: ${event.reason}`;
            }
            if (event.code === 1006) {
                disconnectMessage += `. Abnormal closure.`;
            }
            disconnectMessage += ` Reconnecting in ${RECONNECT_DELAY / 1000}s...`;
            setConnectionStatus(disconnectMessage);

            if (!reconnectIntervalRef.current) {
                reconnectIntervalRef.current = setInterval(() => connectWebSocket(feedId), RECONNECT_DELAY);
                console.log(`[WS: ${feedId}] Started reconnect interval.`);
            }
        };
    }, [db, isFirebaseReady, currentUserId, resolveBlueskyDidAndProfile]);

    const stopWebSocket = useCallback(() => {
        console.log("[stopWebSocket] Attempting to stop current WebSocket and clear interval.");
        if (reconnectIntervalRef.current) {
            clearInterval(reconnectIntervalRef.current);
            reconnectIntervalRef.current = null;
            console.log("[stopWebSocket] Cleared existing reconnectInterval.");
        }
        if (wsRef.current) {
            if (wsRef.current.readyState === WebSocket.OPEN || wsRef.current.readyState === WebSocket.CONNECTING) {
                wsRef.current.close(1000, "Stopping old connection");
                console.log("[stopWebSocket] Closed existing WebSocket with code 1000.");
            } else {
                console.log(`[stopWebSocket] WebSocket not in OPEN/CONNECTING state (readyState: ${wsRef.current.readyState}). Skipping close.`);
            }
            wsRef.current = null;
            console.log("[stopWebSocket] Nullified global wsRef.current reference.");
        }
        currentConnectingFeedIdRef.current = null;
        setConnectionStatus('Idle'); // Set status to Idle when WS is stopped
    }, []);

    // Debounced Firestore Update Logic (memoized)
    const debouncedUpdateFirestoreAnalysis = useCallback((feedId, authorDid, text, embed, postUri, timestamp, facets) => {
        if (!feeds[feedId].analyzeData) {
            console.log(`[Debounce] Skipping data processing for feed: ${feedId} (analyzeData is false).`);
            return;
        }

        if (!accumulatedUpdatesRef.current[feedId]) {
            accumulatedUpdatesRef.current[feedId] = {
                termCounts: {}, linkCounts: {}, domainCounts: {}, contentTypeCounts: {},
                linkCardsData: {}, imageCardsData: {}, posterCounts: {}, posterLikes: {},
                mentionCounts: {}, latestPostTimestamp: null
            };
        }

        const safeText = String(text || '');

        // Hashtag Counting
        const hashtags = (safeText.toLowerCase().match(/#\b\w+\b/g) || []).map(tag => tag.substring(1));
        hashtags.forEach(tag => {
            accumulatedUpdatesRef.current[feedId].termCounts[tag] = (accumulatedUpdatesRef.current[feedId].termCounts[tag] || 0) + 1;
        });

        // Mention Counting
        if (Array.isArray(facets)) {
            facets.forEach(facet => {
                if (facet.features && Array.isArray(facet.features)) {
                    facet.features.forEach(feature => {
                        if (feature.$type === 'app.bsky.richtext.facet#mention' && feature.did) {
                            accumulatedUpdatesRef.current[feedId].mentionCounts[feature.did] = (accumulatedUpdatesRef.current[feedId].mentionCounts[feature.did] || 0) + 1;
                        }
                    });
                }
            });
        }

        const { link: extractedLink, title: extractedTitle, description: extractedDescription, thumbnailUrl: extractedThumbnailUrl } = extractEmbedData(embed);

        if (extractedLink) {
            if (!accumulatedUpdatesRef.current[feedId].linkCounts[extractedLink]) {
                accumulatedUpdatesRef.current[feedId].linkCounts[extractedLink] = { count: 0, headline: null, description: null, originalTitle: null };
            }
            accumulatedUpdatesRef.current[feedId].linkCounts[extractedLink].count += 1;
            if (extractedTitle && accumulatedUpdatesRef.current[feedId].linkCounts[extractedLink].originalTitle === null) {
                accumulatedUpdatesRef.current[feedId].linkCounts[extractedLink].originalTitle = extractedTitle;
            }
            if (extractedDescription && accumulatedUpdatesRef.current[feedId].linkCounts[extractedLink].description === null) {
                accumulatedUpdatesRef.current[feedId].linkCounts[extractedLink].description = extractedDescription;
            }
            fetchHeadline(extractedLink).then(fetchedTitle => {
                if (fetchedTitle && accumulatedUpdatesRef.current[feedId]?.linkCounts[extractedLink]?.headline === null) {
                    accumulatedUpdatesRef.current[feedId].linkCounts[extractedLink].headline = fetchedTitle;
                }
            });

            if (!accumulatedUpdatesRef.current[feedId].linkCardsData[extractedLink]) {
                accumulatedUpdatesRef.current[feedId].linkCardsData[extractedLink] = { count: 0, title: null, description: null, recentPosts: [], thumbnailUrl: null };
            }
            accumulatedUpdatesRef.current[feedId].linkCardsData[extractedLink].count += 1;
            if (extractedTitle && accumulatedUpdatesRef.current[feedId].linkCardsData[extractedLink].title === null) {
                accumulatedUpdatesRef.current[feedId].linkCardsData[extractedLink].title = extractedTitle;
            }
            if (extractedDescription && accumulatedUpdatesRef.current[feedId].linkCardsData[extractedLink].description === null) {
                accumulatedUpdatesRef.current[feedId].linkCardsData[extractedLink].description = extractedDescription;
            }
            if (extractedThumbnailUrl && accumulatedUpdatesRef.current[feedId].linkCardsData[extractedLink].thumbnailUrl === null) {
                accumulatedUpdatesRef.current[feedId].linkCardsData[extractedLink].thumbnailUrl = extractedThumbnailUrl;
            }

            if (postUri && timestamp) {
                const currentPosterPosts = (accumulatedUpdatesRef.current[feedId].posterCounts[authorDid] || 0) + 1;
                const newPost = {
                    postUri: postUri,
                    authorDid: authorDid,
                    textPreview: safeText.substring(0, 150) + (safeText.length > 150 ? '...' : ''),
                    timestamp: timestamp,
                    posterTotalPosts: currentPosterPosts,
                    hashtags: hashtags
                };

                let posts = accumulatedUpdatesRef.current[feedId].linkCardsData[extractedLink].recentPosts;
                const existingPostIndex = posts.findIndex(p => p.postUri === newPost.postUri);
                if (existingPostIndex !== -1) {
                    posts[existingPostIndex] = newPost;
                } else {
                    posts.push(newPost);
                }
                posts.sort((a, b) => (b.posterTotalPosts || 0) - (a.posterTotalPosts || 0));
                accumulatedUpdatesRef.current[feedId].linkCardsData[extractedLink].recentPosts = posts.slice(0, 5);
            }

            const baseDomain = getSimplifiedDomain(extractedLink);
            if (baseDomain && baseDomain !== 'Invalid Domain') {
                const originalHost = new URL(extractedLink).hostname.toLowerCase();
                if (!accumulatedUpdatesRef.current[feedId].domainCounts[baseDomain]) {
                    accumulatedUpdatesRef.current[feedId].domainCounts[baseDomain] = { count: 0, sources: {} };
                }
                accumulatedUpdatesRef.current[feedId].domainCounts[baseDomain].count++;
                accumulatedUpdatesRef.current[feedId].domainCounts[baseDomain].sources[originalHost] = (accumulatedUpdatesRef.current[feedId].domainCounts[baseDomain].sources[originalHost] || 0) + 1;
            }
        } else if (extractedThumbnailUrl) {
            const imageKey = extractedThumbnailUrl;
            if (!accumulatedUpdatesRef.current[feedId].imageCardsData[imageKey]) {
                accumulatedUpdatesRef.current[feedId].imageCardsData[imageKey] = { count: 0, alt: 'Image' };
            }
            accumulatedUpdatesRef.current[feedId].imageCardsData[imageKey].count += 1;
        }

        let contentType = 'Text-only';
        if (embed) {
            if (embed.$type === 'app.bsky.embed.external') contentType = 'Link';
            else if (embed.$type === 'app.bsky.embed.images') contentType = 'Image';
            else if (embed.$type === 'app.bsky.embed.record') contentType = 'Quote/Record';
            else if (embed.$type === 'app.bsky.embed.recordWithMedia') contentType = 'Record with Media';
            else contentType = 'Other Embed';
        }
        accumulatedUpdatesRef.current[feedId].contentTypeCounts[contentType] = (accumulatedUpdatesRef.current[feedId].contentTypeCounts[contentType] || 0) + 1;

        if (authorDid && authorDid !== 'unknown-did') {
            accumulatedUpdatesRef.current[feedId].posterCounts[authorDid] = (accumulatedUpdatesRef.current[feedId].posterCounts[authorDid] || 0) + 1;
            accumulatedUpdatesRef.current[feedId].posterLikes[authorDid] = accumulatedUpdatesRef.current[feedId].posterCounts[authorDid];
        }

        if (!accumulatedUpdatesRef.current[feedId].latestPostTimestamp || new Date(timestamp) > new Date(accumulatedUpdatesRef.current[feedId].latestPostTimestamp)) {
            accumulatedUpdatesRef.current[feedId].latestPostTimestamp = timestamp;
        }

        clearTimeout(debounceTimerRef.current);
        debounceTimerRef.current = setTimeout(async () => {
            console.log(`[Debounce] Executing debounced Firestore update for feed: ${currentActiveFeedId}.`);
            if (!db || !isFirebaseReady) {
                console.warn("Firestore: Not ready for update after debounce. Cannot update analysis.");
                return;
            }

            const feedsToProcess = Object.keys(accumulatedUpdatesRef.current);

            for (const currentFeedIdToProcess of feedsToProcess) {
                if (!feeds[currentFeedIdToProcess].analyzeData) {
                    console.log(`[Debounce] Skipping Firestore write for feed: ${currentFeedIdToProcess} (analyzeData is false).`);
                    continue;
                }

                const feedUpdates = accumulatedUpdatesRef.current[currentFeedIdToProcess];
                const dailyDocDate = new Date(feedUpdates.latestPostTimestamp || new Date());
                const dailyDocId = getYYYYMMDD(dailyDocDate);

                const summaryDocRef = doc(db, 'artifacts', APP_ID, 'public', 'data', 'feeds', currentFeedIdToProcess, 'summaries', 'allTime');
                const dailyDocRef = doc(db, 'artifacts', APP_ID, 'public', 'data', 'feeds', currentFeedIdToProcess, 'dailyAggregates', dailyDocId);

                try {
                    const liveSummarySnap = await getDoc(summaryDocRef);
                    let currentSummaryData = liveSummarySnap.exists() ? liveSummarySnap.data() : {
                        termCounts: {}, linkCounts: {}, domainCounts: {}, contentTypeCounts: {},
                        linkCardsData: {}, imageCardsData: {}, posterCounts: {}, posterLikes: {},
                        mentionCounts: {}
                    };

                    for (const key in feedUpdates.termCounts) {
                        currentSummaryData.termCounts[key] = (currentSummaryData.termCounts[key] || 0) + feedUpdates.termCounts[key];
                    }
                    for (const url in feedUpdates.linkCounts) {
                        const { count, headline, originalTitle, description } = feedUpdates.linkCounts[url];
                        if (typeof currentSummaryData.linkCounts[url] !== 'object' || currentSummaryData.linkCounts[url] === null) {
                            currentSummaryData.linkCounts[url] = { count: 0, headline: null, description: null, originalTitle: null };
                        }
                        currentSummaryData.linkCounts[url].count = (currentSummaryData.linkCounts[url].count || 0) + count;
                        if (headline && currentSummaryData.linkCounts[url].headline === null) {
                            currentSummaryData.linkCounts[url].headline = headline;
                        }
                        if (originalTitle && currentSummaryData.linkCounts[url].originalTitle === null) {
                            currentSummaryData.linkCounts[url].originalTitle = originalTitle;
                        }
                        if (description && currentSummaryData.linkCounts[url].description === null) {
                            currentSummaryData.linkCounts[url].description = description;
                        }
                    }
                    for (const simplifiedDomain in feedUpdates.domainCounts) {
                        const updateData = feedUpdates.domainCounts[simplifiedDomain];
                        if (!currentSummaryData.domainCounts[simplifiedDomain]) {
                            currentSummaryData.domainCounts[simplifiedDomain] = { count: 0, sources: {} };
                        }
                        currentSummaryData.domainCounts[simplifiedDomain].count = (currentSummaryData.domainCounts[simplifiedDomain].count || 0) + updateData.count;
                        for (const originalHost in updateData.sources) {
                            currentSummaryData.domainCounts[simplifiedDomain].sources[originalHost] =
                                (currentSummaryData.domainCounts[simplifiedDomain].sources[originalHost] || 0) + updateData.sources[originalHost];
                        }
                    }
                    for (const key in feedUpdates.contentTypeCounts) {
                        currentSummaryData.contentTypeCounts[key] = (currentSummaryData.contentTypeCounts[key] || 0) + feedUpdates.contentTypeCounts[key];
                    }
                    for (const url in feedUpdates.linkCardsData) {
                        const { count, title, description, recentPosts, thumbnailUrl } = feedUpdates.linkCardsData[url];
                        if (typeof currentSummaryData.linkCardsData[url] !== 'object' || currentSummaryData.linkCardsData[url] === null) {
                            currentSummaryData.linkCardsData[url] = { count: 0, title: null, description: null, recentPosts: [], thumbnailUrl: null };
                        }
                        currentSummaryData.linkCardsData[url].count = (currentSummaryData.linkCardsData[url].count || 0) + count;
                        if (title && currentSummaryData.linkCardsData[url].title === null) {
                            currentSummaryData.linkCardsData[url].title = title;
                        }
                        if (description && currentSummaryData.linkCardsData[url].description === null) {
                            currentSummaryData.linkCardsData[url].description = description;
                        }
                        if (thumbnailUrl && currentSummaryData.linkCardsData[url].thumbnailUrl === null) {
                            currentSummaryData.linkCardsData[url].thumbnailUrl = thumbnailUrl;
                        }

                        let existingRecentPosts = currentSummaryData.linkCardsData[url].recentPosts || [];
                        const newPostsMap = new Map(existingRecentPosts.map(p => [p.postUri, p]));
                        recentPosts.forEach(newPost => newPostsMap.set(newPost.postUri, newPost));
                        existingRecentPosts = Array.from(newPostsMap.values());
                        existingRecentPosts.sort((a, b) => (b.posterTotalPosts || 0) - (a.posterTotalPosts || 0));
                        currentSummaryData.linkCardsData[url].recentPosts = existingRecentPosts.slice(0, 5);
                    }
                    for (const key in feedUpdates.imageCardsData) {
                        const { count, alt } = feedUpdates.imageCardsData[key];
                        if (typeof currentSummaryData.imageCardsData[key] !== 'object' || currentSummaryData.imageCardsData[key] === null) {
                            currentSummaryData.imageCardsData[key] = { count: 0, alt: null };
                        }
                        currentSummaryData.imageCardsData[key].count = (currentSummaryData.imageCardsData[key] || 0) + count;
                        if (alt && currentSummaryData.imageCardsData[key].alt === null) {
                            currentSummaryData.imageCardsData[key].alt = alt;
                        }
                    }
                    for (const didKey in feedUpdates.posterCounts) {
                        currentSummaryData.posterCounts[didKey] = (currentSummaryData.posterCounts[didKey] || 0) + feedUpdates.posterCounts[didKey];
                    }
                    for (const didKey in feedUpdates.posterLikes) {
                        currentSummaryData.posterLikes[didKey] = (currentSummaryData.posterLikes[didKey] || 0) + feedUpdates.posterLikes[didKey];
                    }
                    for (const didKey in feedUpdates.mentionCounts) {
                        currentSummaryData.mentionCounts[didKey] = (currentSummaryData.mentionCounts[didKey] || 0) + feedUpdates.mentionCounts[didKey];
                    }

                    await setDoc(summaryDocRef, currentSummaryData, { merge: true });
                    console.log(`[Firestore Write] Successfully updated All-Time Summary for feed ${currentFeedIdToProcess}.`);

                    const dailySnap = await getDoc(dailyDocRef);
                    let currentDailyData = dailySnap.exists() ? dailySnap.data() : {
                        termCounts: {}, linkCounts: {}, domainCounts: {}, contentTypeCounts: {},
                        linkCardsData: {}, imageCardsData: {}, posterCounts: {}, posterLikes: {},
                        mentionCounts: {}
                    };

                    for (const key in feedUpdates.termCounts) {
                        currentDailyData.termCounts[key] = (currentDailyData.termCounts[key] || 0) + feedUpdates.termCounts[key];
                    }
                    for (const url in feedUpdates.linkCounts) {
                        const { count, headline, originalTitle, description } = feedUpdates.linkCounts[url];
                        if (typeof currentDailyData.linkCounts[url] !== 'object' || currentDailyData.linkCounts[url] === null) {
                            currentDailyData.linkCounts[url] = { count: 0, headline: null, description: null, originalTitle: null };
                        }
                        currentDailyData.linkCounts[url].count = (currentDailyData.linkCounts[url].count || 0) + count;
                        if (headline && currentDailyData.linkCounts[url].headline === null) {
                            currentDailyData.linkCardsData[url].headline = headline;
                        }
                        if (originalTitle && currentDailyData.linkCardsData[url].originalTitle === null) {
                            currentDailyData.linkCardsData[url].originalTitle = originalTitle;
                        }
                        if (description && currentDailyData.linkCardsData[url].description === null) {
                            currentDailyData.linkCardsData[url].description = description;
                        }
                    }
                    for (const simplifiedDomain in feedUpdates.domainCounts) {
                        const dailyEntry = feedUpdates.domainCounts[simplifiedDomain];
                        if (!currentDailyData.domainCounts[simplifiedDomain]) {
                            currentDailyData.domainCounts[simplifiedDomain] = { count: 0, sources: {} };
                        }
                        currentDailyData.domainCounts[simplifiedDomain].count += dailyEntry.count;
                        for (const originalHost in dailyEntry.sources) {
                            currentDailyData.domainCounts[simplifiedDomain].sources[originalHost] =
                                (currentDailyData.domainCounts[simplifiedDomain].sources[originalHost] || 0) + dailyEntry.sources[originalHost];
                        }
                    }
                    for (const type in feedUpdates.contentTypeCounts) {
                        currentDailyData.contentTypeCounts[type] = (currentDailyData.contentTypeCounts[type] || 0) + feedUpdates.contentTypeCounts[type];
                    }
                    for (const url in feedUpdates.linkCardsData) {
                        const { count, title, description, recentPosts, thumbnailUrl } = feedUpdates.linkCardsData[url];
                        if (typeof currentDailyData.linkCardsData[url] !== 'object' || currentDailyData.linkCardsData[url] === null) {
                            currentDailyData.linkCardsData[url] = { count: 0, title: null, description: null, recentPosts: [], thumbnailUrl: null };
                        }
                        currentDailyData.linkCardsData[url].count = (currentDailyData.linkCardsData[url].count || 0) + count;
                        if (title && currentDailyData.linkCardsData[url].title === null) {
                            currentDailyData.linkCardsData[url].title = title;
                        }
                        if (description && currentDailyData.linkCardsData[url].description === null) {
                            currentDailyData.linkCardsData[url].description = description;
                        }
                        if (thumbnailUrl && currentDailyData.linkCardsData[url].thumbnailUrl === null) {
                            currentDailyData.linkCardsData[url].thumbnailUrl = thumbnailUrl;
                        }

                        let existingDailyRecentPosts = currentDailyData.linkCardsData[url].recentPosts || [];
                        const newDailyPostsMap = new Map(existingDailyRecentPosts.map(p => [p.postUri, p]));
                        (recentPosts || []).forEach(newPost => newDailyPostsMap.set(newPost.postUri, newPost));
                        existingDailyRecentPosts = Array.from(newDailyPostsMap.values());
                        existingDailyRecentPosts.sort((a, b) => (b.posterTotalPosts || 0) - (a.posterTotalPosts || 0));
                        currentDailyData.linkCardsData[url].recentPosts = existingDailyRecentPosts.slice(0, 5);
                    }
                    for (const key in feedUpdates.imageCardsData) {
                        const { count, alt } = feedUpdates.imageCardsData[key];
                        if (typeof currentDailyData.imageCardsData[key] !== 'object' || currentDailyData.imageCardsData[key] === null) {
                            currentDailyData.imageCardsData[key] = { count: 0, alt: null };
                        }
                        currentDailyData.imageCardsData[key].count = (currentDailyData.imageCardsData[key] || 0) + count;
                        if (alt && currentDailyData.imageCardsData[key].alt === null) {
                            currentDailyData.imageCardsData[key].alt = alt;
                        }
                    }
                    for (const didKey in feedUpdates.posterCounts) {
                        currentDailyData.posterCounts[didKey] = (currentDailyData.posterCounts[didKey] || 0) + feedUpdates.posterCounts[didKey];
                    }
                    for (const didKey in feedUpdates.posterLikes) {
                        currentDailyData.posterLikes[didKey] = (currentDailyData.posterLikes[didKey] || 0) + feedUpdates.posterLikes[didKey];
                    }
                    for (const didKey in feedUpdates.mentionCounts) {
                        currentDailyData.mentionCounts[didKey] = (currentDailyData.mentionCounts[didKey] || 0) + feedUpdates.mentionCounts[didKey];
                    }

                    await setDoc(dailyDocRef, currentDailyData, { merge: true });
                    console.log(`[Firestore Write] Successfully updated Daily Aggregate for feed ${currentFeedIdToProcess} (${dailyDocId}).`);

                } catch (e) {
                    console.error(`[Firestore Write] Error updating Firestore analysis for feed ${currentFeedIdToProcess} (debounced):`, e);
                }
            }

            accumulatedUpdatesRef.current = {}; // Reset all accumulated updates after successful write
            // Re-fetch currentFeedData for the active feed to ensure UI reflects latest state
            fetchFeedSummaryData(currentActiveFeedId);

        }, DEBOUNCE_DELAY);
    }, [db, isFirebaseReady, currentUserId, currentActiveFeedId, fetchFeedSummaryData, resolveBlueskyDidAndProfile]);

    // Effect for initial data load and WebSocket connection after Firebase is ready
    useEffect(() => {
        if (isFirebaseReady && db && auth) {
            fetchResolvedDids();
            fetchFeedSummaryData(currentActiveFeedId);
            if (feeds[currentActiveFeedId].analyzeData) {
                connectWebSocket(currentActiveFeedId);
            } else {
                setConnectionStatus(`No real-time data needed for ${feeds[currentActiveFeedId].displayName}`);
            }
        }
    }, [isFirebaseReady, db, auth, currentActiveFeedId, fetchResolvedDids, fetchFeedSummaryData, connectWebSocket]);

    // Cleanup WebSocket on component unmount
    useEffect(() => {
        return () => {
            stopWebSocket();
        };
    }, [stopWebSocket]);

    // Function to render analysis data based on current state
    const renderAnalysis = useCallback(() => {
        const feed = feeds[currentActiveFeedId];
        const displayData = {
            termCounts: {}, linkCardsData: {}, domainCounts: {}, contentTypeCounts: {},
            posterCounts: {}, posterLikes: {}, mentionCounts: {}
        };

        if (currentSelectedTimeframe === 'allTime') {
            Object.assign(displayData, currentFeedData);
        } else {
            // Aggregate daily data for selected timeframe
            const today = new Date();
            let startDate;

            if (currentSelectedTimeframe === 'day') {
                startDate = new Date(today.getFullYear(), today.getMonth(), today.getDate());
            } else if (currentSelectedTimeframe === 'week') {
                startDate = new Date(today.getFullYear(), today.getMonth(), today.getDate() - today.getDay());
            } else if (currentSelectedTimeframe === 'month') {
                startDate = new Date(today.getFullYear(), today.getMonth(), 1);
            }
            const startYYYYMMDD = getYYYYMMDD(startDate);

            // Fetch daily aggregates for the selected timeframe
            // This is currently a simplified aggregation for UI, ideally this would be done by a backend service
            // to avoid re-fetching all daily documents every time.
            const fetchAndAggregateDailyData = async () => {
                if (!db || !currentUserId) return;
                const dailyAggregatesRef = collection(db, 'artifacts', APP_ID, 'public', 'data', 'feeds', currentActiveFeedId, 'dailyAggregates');
                const q = query(dailyAggregatesRef, where('__name__', '>=', startYYYYMMDD));

                try {
                    const querySnapshot = await getDocs(q);
                    const aggregatedDailyData = {
                        termCounts: {}, linkCardsData: {}, domainCounts: {}, contentTypeCounts: {},
                        posterCounts: {}, posterLikes: {}, mentionCounts: {}
                    };

                    querySnapshot.forEach((docSnap) => {
                        const data = docSnap.data();
                        for (const term in data.termCounts) {
                            aggregatedDailyData.termCounts[term] = (aggregatedDailyData.termCounts[term] || 0) + data.termCounts[term];
                        }
                        for (const url in data.linkCardsData) {
                            const { count, title, description, recentPosts, thumbnailUrl } = data.linkCardsData[url];
                            if (!aggregatedDailyData.linkCardsData[url]) {
                                aggregatedDailyData.linkCardsData[url] = { count: 0, title: null, description: null, recentPosts: [], thumbnailUrl: null };
                            }
                            aggregatedDailyData.linkCardsData[url].count += count;
                            if (title && aggregatedDailyData.linkCardsData[url].title === null) aggregatedDailyData.linkCardsData[url].title = title;
                            if (description && aggregatedDailyData.linkCardsData[url].description === null) aggregatedDailyData.linkCardsData[url].description = description;
                            if (thumbnailUrl && aggregatedDailyData.linkCardsData[url].thumbnailUrl === null) aggregatedDailyData.linkCardsData[url].thumbnailUrl = thumbnailUrl;

                            let existingRecentPosts = aggregatedDailyData.linkCardsData[url].recentPosts || [];
                            const newPostsMap = new Map(existingRecentPosts.map(p => [p.postUri, p]));
                            (recentPosts || []).forEach(newPost => newPostsMap.set(newPost.postUri, newPost));
                            existingRecentPosts = Array.from(newPostsMap.values());
                            existingRecentPosts.sort((a, b) => (b.posterTotalPosts || 0) - (a.posterTotalPosts || 0));
                            aggregatedDailyData.linkCardsData[url].recentPosts = existingRecentPosts.slice(0, 5);
                        }
                        for (const simplifiedDomain in data.domainCounts) {
                            const dailyEntry = data.domainCounts[simplifiedDomain];
                            if (!aggregatedDailyData.domainCounts[simplifiedDomain]) {
                                aggregatedDailyData.domainCounts[simplifiedDomain] = { count: 0, sources: {} };
                            }
                            aggregatedDailyData.domainCounts[simplifiedDomain].count += dailyEntry.count;
                            for (const originalHost in dailyEntry.sources) {
                                aggregatedDailyData.domainCounts[simplifiedDomain].sources[originalHost] =
                                    (aggregatedDailyData.domainCounts[simplifiedDomain].sources[originalHost] || 0) + dailyEntry.sources[originalHost];
                            }
                        }
                        for (const type in data.contentTypeCounts) {
                            aggregatedDailyData.contentTypeCounts[type] = (aggregatedDailyData.contentTypeCounts[type] || 0) + data.contentTypeCounts[type];
                        }
                        for (const didKey in data.posterCounts) {
                            aggregatedDailyData.posterCounts[didKey] = (aggregatedDailyData.posterCounts[didKey] || 0) + data.posterCounts[didKey];
                        }
                        for (const didKey in data.posterLikes) {
                            aggregatedDailyData.posterLikes[didKey] = (aggregatedDailyData.posterLikes[didKey] || 0) + data.posterLikes[didKey];
                        }
                        for (const didKey in data.mentionCounts) {
                            aggregatedDailyData.mentionCounts[didKey] = (aggregatedDailyData.mentionCounts[didKey] || 0) + data.mentionCounts[didKey];
                        }
                    });
                    // Only update the display data, not the main currentFeedData
                    setCurrentFeedData(prevData => ({ ...prevData, ...aggregatedDailyData })); // This will trigger re-render of rankings
                    setConnectionStatus(`Connected to ${feeds[currentActiveFeedId].displayName}`);

                } catch (error) {
                    console.error(`Error fetching ${currentSelectedTimeframe} data for ${currentActiveFeedId}:`, error);
                    setConnectionStatus(`Error fetching ${currentSelectedTimeframe} data for ${feeds[currentActiveFeedId].displayName}: ${error.message}`);
                }
            };
            // Only fetch if Firebase is ready and it's an analyzing feed
            if (db && currentUserId && feeds[currentActiveFeedId].analyzeData) {
                fetchAndAggregateDailyData();
                setConnectionStatus(`Fetching data for ${currentSelectedTimeframe}...`);
            } else if (!feeds[currentActiveFeedId].analyzeData) {
                setConnectionStatus(`No real-time data for ${feeds[currentActiveFeedId].displayName}`);
            }
        }
    }, [currentSelectedTimeframe, currentActiveFeedId, currentFeedData, db, currentUserId]); // Depend on currentFeedData to trigger re-render

    // Effect to re-render analysis when currentFeedData or timeframe changes
    useEffect(() => {
        // This effect runs whenever currentFeedData or currentSelectedTimeframe changes.
        // The renderAnalysis function is responsible for determining what to display.
        if (feeds[currentActiveFeedId].analyzeData || currentActiveFeedId === 'datamaster') {
            renderAnalysis();
        }
    }, [currentFeedData, currentSelectedTimeframe, currentActiveFeedId, renderAnalysis]);


    // Feed switching logic
    const switchFeed = useCallback((newFeedId) => {
        if (currentActiveFeedId === newFeedId) {
            console.log(`Already on ${newFeedId} feed.`);
            return;
        }

        console.log(`Switching feed from ${currentActiveFeedId} to ${newFeedId}`);
        setCurrentActiveFeedId(newFeedId);
        localStorage.setItem('activeFeedId', newFeedId);

        if (feeds[newFeedId].analyzeData) {
            connectWebSocket(newFeedId);
        } else {
            stopWebSocket();
            setConnectionStatus(`No real-time data for ${feeds[newFeedId].displayName}`);
        }
        fetchFeedSummaryData(newFeedId); // Fetch data for the new feed
    }, [currentActiveFeedId, connectWebSocket, stopWebSocket, fetchFeedSummaryData]);

    // Function to render feed version buttons
    const renderFeedVersionButtons = useCallback(() => {
        const feed = feeds[currentActiveFeedId];
        if (!feed || !feed.versions || feed.versions.length <= 1) {
            return null; // No buttons if no multiple versions
        }

        const activeVersionKey = `activeVersion_${currentActiveFeedId}`;
        const currentActiveVersionSrc = localStorage.getItem(activeVersionKey) || feed.versions[0].src;

        return feed.versions.map((version) => (
            <button
                key={version.name}
                className={`px-4 py-2 text-sm font-medium rounded-md transition duration-200 ${
                    version.src === currentActiveVersionSrc
                        ? 'bg-blue-500 text-white hover:bg-blue-600'
                        : 'bg-gray-200 text-gray-700 hover:bg-gray-300 dark:bg-gray-700 dark:text-gray-200 dark:hover:bg-gray-600'
                }`}
                onClick={() => {
                    document.getElementById('mainIframe').src = version.src;
                    localStorage.setItem(activeVersionKey, version.src);
                    // Force re-render of this component to update button active state
                    setCurrentActiveFeedId(currentActiveFeedId); 
                }}
            >
                {version.name}
            </button>
        ));
    }, [currentActiveFeedId]);

    // Function to handle opening link posts modal
    const openLinkPostsModal = useCallback((url) => {
        const linkData = currentFeedData.linkCardsData[url];
        if (!linkData || !linkData.recentPosts || linkData.recentPosts.length === 0) {
            setLinkPostsModalContent({
                title: `No posts found for: ${linkData?.title || url}`,
                posts: []
            });
        } else {
            const sortedPosts = [...linkData.recentPosts].sort((a, b) => (b.posterTotalPosts || 0) - (a.posterTotalPosts || 0));
            const postsWithResolvedInfo = sortedPosts.map(post => {
                const resolvedInfo = resolvedDidsCache[post.authorDid] || { handle: '[unknown]', displayName: '[unknown]' };
                return { ...post, resolvedInfo };
            });
            setLinkPostsModalContent({
                title: `Top Posts for: ${linkData.title || url}`,
                posts: postsWithResolvedInfo
            });
        }
        setLinkPostsModalOpen(true);
    }, [currentFeedData.linkCardsData, resolvedDidsCache]);


    // Determine the current feed to display based on currentActiveFeedId
    const activeFeed = feeds[currentActiveFeedId];
    const feedTitleIconSrc = activeFeed?.icon.startsWith('http') ? activeFeed.icon : null;
    const feedTitleIconSvg = activeFeed?.icon === 'train' ? (
        <svg className="w-full h-full" fill="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
            <path d="M12 2C6.477 2 2 6.477 2 12s4.477 10 10 10 10-4.477 10-10S17.523 2 12 2zm-1 14H7.5c-.276 0-.5-.224-.5-.5V7.5c0-.276.224-.5.5-.5H11V16zm5.5 0H13c-.276 0-.5-.224-.5-.5V7.5c0-.276.224-.5.5-.5H16.5c.276 0 .5.224.5.5V15.5c0 .276-.224.5-.5.5z"/>
        </svg>
    ) : activeFeed?.icon === 'home' ? (
        <svg className="w-full h-full" fill="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
            <path d="M10 20v-6h4v6h5v-8h3L12 3 2 12h3v8z"></path>
        </svg>
    ) : null;

    const connectionStatusClass = connectionStatus.includes('Connected') ? 'text-green-600' :
                                  connectionStatus.includes('Connecting') || connectionStatus.includes('Fetching') || connectionStatus.includes('Reconnecting') ? 'text-orange-600' :
                                  connectionStatus.includes('Error') || connectionStatus.includes('Disconnected') ? 'text-red-600' : 'text-gray-600';


    // Data for rendering rankings
    const termsToDisplay = currentFeedData.termCounts;
    const linksToDisplay = currentFeedData.linkCardsData;
    const domainsToDisplay = currentFeedData.domainCounts;
    const contentTypesToDisplay = currentFeedData.contentTypeCounts;
    const postersToDisplay = currentFeedData.posterCounts;
    const mentionsToDisplay = currentFeedData.mentionCounts;

    const sortedTerms = Object.entries(termsToDisplay || {})
        .sort(([, countA], [, countB]) => countB - countA)
        .slice(0, 20);

    const sortedRichLinkCards = Object.entries(linksToDisplay || {})
        .sort(([, dataA], [, dataB]) => dataB.count - dataA.count)
        .slice(0, 10);

    const canonicalizedDomainData = canonicalizeDisplayDomains(domainsToDisplay || {});
    const sortedDomains = Object.entries(canonicalizedDomainData)
        .sort(([, dataA], [, dataB]) => dataB.count - dataA.count)
        .slice(0, 10);

    const sortedContentTypes = Object.entries(contentTypesToDisplay || {})
        .sort(([, countA], [, countB]) => countB - countA)
        .slice(0, 5);

    const sortedPosters = Object.entries(postersToDisplay || {})
        .sort(([, countA], [, countB]) => countB - countA)
        .slice(0, 10);

    const sortedMentions = Object.entries(mentionsToDisplay || {})
        .sort(([, countA], [, countB]) => countB - countA)
        .slice(0, 10);

    return (
        <div className="main-wrapper">
            <div className="section-wrapper">
                <div className="section-card flex flex-col justify-between items-center p-3 h-full">
                    <a id="femaProfileBtn" href="https://bsky.app/profile/fema.monster" target="_blank" rel="noopener noreferrer"
                       className="p-0 bg-transparent rounded-full shadow-lg transition-all duration-300 ease-in-out transform hover:scale-105 focus:outline-none focus:ring-2 focus:ring-blue-400 focus:ring-opacity-75">
                        <img src="https://raw.githubusercontent.com/femavibes/datamaster/refs/heads/main/fema.jpg" alt="Fema Monster" className="w-12 h-12 rounded-full border-2 border-transparent hover:border-blue-500 transition-all duration-300"/>
                    </a>
                    <div className="line-spacer"></div>

                    <button id="homeFeedBtn" className={`feed-select-btn bg-gray-200 mb-4 ${currentActiveFeedId === 'datamaster' ? 'active-feed-btn' : ''}`}
                            onClick={() => switchFeed('datamaster')}>
                        <svg className="w-6 h-6" fill="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
                            <path d="M10 20v-6h4v6h5v-8h3L12 3 2 12h3v8z"></path>
                        </svg>
                    </button>
                    <div className="line-spacer"></div>

                    <button id="plusBtn" className="p-3 bg-blue-500 hover:bg-blue-600 text-white rounded-full shadow-lg transition-all duration-300 ease-in-out transform hover:scale-105 focus:outline-none focus:ring-2 focus:ring-blue-400 focus:ring-opacity-75"
                            onClick={() => setPlusButtonModalOpen(true)}>
                        <svg className="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
                            <path strokeLinecap="round" strokeLinejoin="round" strokeWidth="2" d="M12 6v6m0 0v6m0-6h6m-6 0H6"></path>
                        </svg>
                    </button>
                    <div className="line-spacer"></div>

                    <div className="nav-buttons-container">
                        <button id="urbanismPlusFeedBtn" className={`feed-select-btn bg-gray-200 ${currentActiveFeedId === 'urbanismPlus' ? 'active-feed-btn' : ''}`}
                                onClick={() => switchFeed('urbanismPlus')}>
                            <img src="https://raw.githubusercontent.com/femavibes/datamaster/refs/heads/main/uplus.jpg" alt="Urbanism+ Logo"/>
                        </button>
                        <button id="bandcampFeedBtn" className={`feed-select-btn bg-gray-200 ${currentActiveFeedId === 'bandcamp' ? 'active-feed-btn' : ''}`}
                                onClick={() => switchFeed('bandcamp')}>
                            <img src="https://raw.githubusercontent.com/femavibes/datamaster/refs/heads/main/bandcamp7796.logowik.com.webp" alt="Bandcamp Logo"/>
                        </button>
                        <button id="transitSkyFeedBtn" className={`feed-select-btn bg-gray-200 ${currentActiveFeedId === 'transitSky' ? 'active-feed-btn' : ''}`}
                                onClick={() => switchFeed('transitSky')}>
                            <svg className="w-6 h-6" fill="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
                                <path d="M12 2C6.477 2 2 6.477 2 12s4.477 10 10 10 10-4.477 10-10S17.523 2 12 2zm-1 14H7.5c-.276 0-.5-.224-.5-.5V7.5c0-.276.224-.5.5-.5H11V16zm5.5 0H13c-.276 0-.5-.224-.5-.5V7.5c0-.276.224-.5.5-.5H16.5c.276 0 .5.224.5.5V15.5c0 .276-.224.5-.5.5z"/>
                            </svg>
                        </button>
                    </div>

                    <button id="openSettingsBtn" className="p-3 bg-blue-500 hover:bg-blue-600 text-white rounded-full shadow-lg transition-all duration-300 ease-in-out transform hover:scale-105 focus:outline-none focus:ring-2 focus:ring-blue-400 focus:ring-opacity-75"
                            onClick={() => setSettingsModalOpen(true)}>
                        <svg className="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
                            <path strokeLinecap="round" strokeLinejoin="round" strokeWidth="2" d="M10.325 4.317c.426-1.756 2.924-1.756 3.35 0a1.724 1.724 0 002.573 1.066c1.543-.942 3.333.942 1.89 2.395a1.724 1.724 0 00.01 2.59c1.543 1.453-.294 3.333-1.89 2.395a1.724 1.724 0 00-2.573 1.066c-.426 1.756-2.924 1.756-3.35 0a1.724 1.724 0 00-2.573-1.066c-1.543.942-3.333-.942-1.89-2.395a1.724 1.724 0 00-.01-2.59c-1.543-1.453.294-3.333 1.89-2.395a1.724 1.724 0 002.573-1.066z"></path>
                            <path strokeLinecap="round" strokeLinejoin="round" strokeWidth="2" d="M15 12a3 3 0 11-6 0 3 3 0 016 0z"></path>
                        </svg>
                    </button>
                </div>
            </div>

            <div className="section-wrapper">
                <div className="section-card">
                    <div className="flex justify-between items-center mb-4">
                        <h2 className="stylish-section-title mb-0 mr-4">
                            <a id="feedTitleLink" href={activeFeed?.blueskyProfileUrl} target="_blank" rel="noopener noreferrer" className="feed-title-link">
                                <div className="feed-title-icon-wrapper">
                                    {feedTitleIconSrc && <img src={feedTitleIconSrc} alt="Feed Logo" style={{ '--title-icon-scale': activeFeed?.titleIconScale || '1' }} />}
                                    {feedTitleIconSvg}
                                </div>
                                <span id="feedTitleText">{activeFeed?.displayName}</span>
                            </a>
                        </h2>
                        <div id="feedVersionButtons" className="flex gap-2">
                            {renderFeedVersionButtons()}
                        </div>
                    </div>
                    <iframe
                        id="mainIframe"
                        src={localStorage.getItem(`activeVersion_${currentActiveFeedId}`) || activeFeed?.versions[0]?.src}
                        title="Social Feed"
                        className="iframe-embed"
                        allowFullScreen
                    ></iframe>
                </div>
            </div>

            <div className="section-wrapper">
                <div className="section-card">
                    <div className="flex flex-col sm:flex-row justify-between sm:items-center w-full mb-4">
                        <div className="flex flex-col sm:flex-row items-center flex-grow-0 gap-4 mb-4 sm:mb-0">
                            <h2 className="stylish-section-title mb-0" id="rankingsSectionTitle">
                                {currentActiveFeedId === 'datamaster' ? 'Discover Feeds' : 'Rankings'}
                            </h2>
                            <div id="timeframeSelectorContainer" className={`timeframe-selector flex-shrink-0 sm:ml-8 ${currentActiveFeedId === 'datamaster' ? 'hidden' : 'flex'}`}>
                                {['day', 'week', 'month', 'allTime'].map(timeframe => (
                                    <React.Fragment key={timeframe}>
                                        <input
                                            type="radio"
                                            id={`timeframe${timeframe.charAt(0).toUpperCase() + timeframe.slice(1)}`}
                                            name="timeframe"
                                            value={timeframe}
                                            checked={currentSelectedTimeframe === timeframe}
                                            onChange={(e) => setCurrentSelectedTimeframe(e.target.value)}
                                        />
                                        <label htmlFor={`timeframe${timeframe.charAt(0).toUpperCase() + timeframe.slice(1)}`}>{timeframe.charAt(0).toUpperCase() + timeframe.slice(1)}</label>
                                    </React.Fragment>
                                ))}
                            </div>
                        </div>
                        <div className="text-sm text-gray-600 flex flex-col items-end flex-shrink-0">
                            <span id="connectionStatus" className={connectionStatusClass}>{connectionStatus}</span>
                            <span id="datamasterVersionText" className="text-gray-600">datamaster 0.8.8</span>
                        </div>
                    </div>

                    {/* Live Posts Container (hidden) */}
                    <div id="postsContainer" className="flex-grow overflow-y-auto border border-gray-200 dark:border-gray-700 rounded-md p-2" style={{ maxHeight: '500px', display: 'none' }}>
                        {postsToDisplay.length === 0 ? (
                            <p id="noPostsMessage" className="text-gray-500 text-center py-4">Waiting for new posts...</p>
                        ) : (
                            postsToDisplay.map((post, index) => (
                                <div key={index} className="post-card">
                                    <div className="post-author">
                                        <div className="text-gray-700">{post.displayName}</div>
                                        <div className="text-blue-600 text-sm">{post.handle}</div>
                                    </div>
                                    <div className="post-content">{post.content}</div>
                                    <div className="post-timestamp">{post.timestamp}</div>
                                </div>
                            ))
                        )}
                    </div>

                    {/* Rankings Content (Conditional display) */}
                    <div id="rankingsContent" className={`grid grid-cols-1 md:grid-cols-12 gap-6 mt-6 w-full flex-grow ${currentActiveFeedId === 'datamaster' ? 'hidden' : 'grid'}`}>
                        <div className="md:col-span-3 flex flex-col" id="topHashtagsSection">
                            <h3 className="stylish-subheader">Top Hashtags</h3>
                            <div className="ranking-scroll-area">
                                {sortedTerms.length === 0 ? (
                                    <p className="text-gray-500 text-center">No hashtags yet to analyze.</p>
                                ) : (
                                    sortedTerms.map(([term, count], index) => (
                                        <div key={term} className="domain-card" title={term}>
                                            <div className="domain-card-name">
                                                {index + 1}. {term}
                                            </div>
                                            <div className="domain-card-count">
                                                {count} uses
                                            </div>
                                        </div>
                                    ))
                                )}
                            </div>
                        </div>
                        <div className="md:col-span-5 flex flex-col" id="topLinksSection">
                            <h3 className="stylish-subheader">Top Links</h3>
                            <div className="ranking-scroll-area">
                                {sortedRichLinkCards.length === 0 ? (
                                    <p className="text-gray-500 text-center">No rich link cards yet to display.</p>
                                ) : (
                                    sortedRichLinkCards.map(([url, data]) => (
                                        <a key={url} href={url} target="_blank" rel="noopener noreferrer" className="link-card">
                                            {data.thumbnailUrl ? (
                                                <img src={data.thumbnailUrl} onError={(e) => { e.target.onerror = null; e.target.src = 'https://placehold.co/128x128/e2e8f0/333?text=No+Image'; e.target.alt = 'No Image Available'; }} alt={data.title || 'Link thumbnail'} className="w-full h-32 object-cover rounded-t-md mb-2" />
                                            ) : (
                                                <img src="https://placehold.co/128x128/e2e8f0/333?text=No+Image" alt="No Image Available" className="w-full h-32 object-cover rounded-t-md mb-2" />
                                            )}
                                            <div className="link-card-headline">
                                                {data.title || 'No Title Available'}
                                            </div>
                                            <div className="link-card-description">{data.description || 'No description available.'}</div>
                                            <div className="link-card-url">{url}</div>
                                            <div className="link-card-footer">
                                                <span className="link-card-count">{data.count} shares</span>
                                                <button type="button" className="link-card-comment-btn" title={`View posts for ${url}`} onClick={(e) => { e.preventDefault(); openLinkPostsModal(url); }}>
                                                    <svg fill="currentColor" viewBox="0 0 20 20" xmlns="http://www.w3.org/2000/svg">
                                                        <path fillRule="evenodd" d="M18 10c0 3.866-3.582 7-8 7a8.841 8.841 0 01-4.735-1.579l-3.321 1.096A1 1 0 01.328 15.22l1.096-3.321A7.002 7.002 0 002 10c0-3.866 3.582-7 8-7s8 3.134 8 7z" clipRule="evenodd"></path>
                                                    </svg>
                                                </button>
                                            </div>
                                        </a>
                                    ))
                                )}
                            </div>
                        </div>
                        <div className="md:col-span-4 flex flex-col" id="rankedUsersSection">
                            <div className="flex flex-col flex-grow">
                                <div className="flex items-center justify-between mb-4 flex-wrap">
                                    <h3 className="stylish-subheader flex-grow mb-0 mr-2">Ranked Users</h3>
                                    <div className="flex gap-2 flex-shrink-0">
                                        <button className={`ranking-toggle-btn ${currentPosterMentionsView === 'posters' ? 'active' : ''}`}
                                                onClick={() => setCurrentPosterMentionsView('posters')}>Posters</button>
                                        <button className={`ranking-toggle-btn ${currentPosterMentionsView === 'mentions' ? 'active' : ''}`}
                                                onClick={() => setCurrentPosterMentionsView('mentions')}>Mentions</button>
                                    </div>
                                </div>
                                <div className={`ranking-scroll-area ${currentPosterMentionsView === 'posters' ? '' : 'hidden'}`}>
                                    {sortedPosters.length === 0 ? (
                                        <p className="text-gray-500 text-center">No posters yet to rank from the feed.</p>
                                    ) : (
                                        sortedPosters.map(([did, count]) => {
                                            const resolvedInfo = resolvedDidsCache[did] || { handle: '[unknown]', displayName: '[unknown]', avatar: 'https://placehold.co/40x40/cccccc/333?text=?' };
                                            return (
                                                <a key={did} href={`https://bsky.app/profile/${resolvedInfo.handle}`} target="_blank" rel="noopener noreferrer" className="poster-card" title={`DID: ${did}\nHandle: ${resolvedInfo.handle}\nDisplay Name: ${resolvedInfo.displayName}`}>
                                                    <img src={resolvedInfo.avatar} onError={(e) => { e.target.onerror = null; e.target.src = 'https://placehold.co/40x40/cccccc/333?text=?'; }} alt={`${resolvedInfo.displayName}'s avatar`} className="poster-avatar"/>
                                                    <div className="poster-info">
                                                        <div className="poster-display-name">{resolvedInfo.displayName}</div>
                                                        <div className="poster-handle">@{resolvedInfo.handle}</div>
                                                    </div>
                                                    <div className="poster-count">
                                                        {count} posts
                                                    </div>
                                                </a>
                                            );
                                        })
                                    )}
                                </div>
                                <div className={`ranking-scroll-area ${currentPosterMentionsView === 'mentions' ? '' : 'hidden'}`}>
                                    {sortedMentions.length === 0 ? (
                                        <p className="text-gray-500 text-center">No mentions yet to rank from the feed.</p>
                                    ) : (
                                        sortedMentions.map(([did, count]) => {
                                            const resolvedInfo = resolvedDidsCache[did] || { handle: '[unknown]', displayName: '[unknown]', avatar: 'https://placehold.co/40x40/cccccc/333?text=?m' };
                                            return (
                                                <a key={did} href={`https://bsky.app/profile/${resolvedInfo.handle}`} target="_blank" rel="noopener noreferrer" className="poster-card" title={`DID: ${did}\nHandle: ${resolvedInfo.handle}\nDisplay Name: ${resolvedInfo.displayName}`}>
                                                    <img src={resolvedInfo.avatar} onError={(e) => { e.target.onerror = null; e.target.src = 'https://placehold.co/40x40/cccccc/333?text=?'; }} alt={`${resolvedInfo.displayName}'s avatar`} className="poster-avatar"/>
                                                    <div className="poster-info">
                                                        <div className="poster-display-name">{resolvedInfo.displayName}</div>
                                                        <div className="poster-handle">@{resolvedInfo.handle}</div>
                                                    </div>
                                                    <div className="poster-count">
                                                        {count} mentions
                                                    </div>
                                                </a>
                                            );
                                        })
                                    )}
                                </div>
                            </div>
                        </div>

                        {/* Additional sections, hidden by default as per old HTML */}
                        <div className="md:col-span-4 flex flex-col" id="topSitesSection" style={{ display: 'none' }}>
                            <h3 className="stylish-subheader">Top Sites</h3>
                            <div className="ranking-scroll-area">
                                {sortedDomains.length === 0 ? (
                                    <p className="text-gray-500 text-center">No domains yet to analyze.</p>
                                ) : (
                                    sortedDomains.map(([domain, data]) => (
                                        <div key={domain} className="domain-card" title={`Website: ${domain}\nIncludes: ${Array.from(data.original_full_domains_list).join(', ')}`}>
                                            <div className="domain-card-name">{domain}</div>
                                            <div className="domain-card-count">{data.count} visits</div>
                                        </div>
                                    ))
                                )}
                            </div>
                        </div>
                        <div className="md:col-span-5 flex flex-col" id="contentTypeBreakdownSection" style={{ display: 'none' }}>
                            <h3 className="stylish-subheader">Content Type Breakdown</h3>
                            <div className="ranking-scroll-area">
                                {sortedContentTypes.length === 0 ? (
                                    <p className="text-gray-500 text-center">No content types yet to analyze.</p>
                                ) : (
                                    sortedContentTypes.map(([type, count]) => (
                                        <div key={type} className="domain-card">
                                            <div className="domain-card-name">{type}</div>
                                            <div className="domain-card-count">{count} posts</div>
                                        </div>
                                    ))
                                )}
                            </div>
                        </div>
                    </div>

                    {/* Discover Feeds Content (Conditional display) */}
                    <div id="featuredFeedsContent" className={`grid grid-cols-1 md:grid-cols-12 gap-6 mt-6 w-full ${currentActiveFeedId === 'datamaster' ? 'grid' : 'hidden'}`}>
                        <h3 className="md:col-span-12 text-center text-gray-600 dark:text-gray-400">Discover other curated feeds!</h3>
                        <a href="#" className="featured-feed-card md:col-span-4" onClick={() => switchFeed('urbanismPlus')}>
                            <img src="https://raw.githubusercontent.com/femavibes/datamaster/refs/heads/main/uplus.jpg" alt="Urbanism+ Logo" className="featured-feed-icon"/>
                            <div className="featured-feed-info">
                                <div className="featured-feed-name">Urbanism+</div>
                                <div className="featured-feed-description">Daily news and insights on urban planning, public transit, and walkable cities.</div>
                            </div>
                        </a>
                        <a href="#" className="featured-feed-card md:col-span-4" onClick={() => switchFeed('bandcamp')}>
                            <img src="https://raw.githubusercontent.com/femavibes/datamaster/refs/heads/main/bandcamp7796.logowik.com.webp" alt="Bandcamp Logo" className="featured-feed-icon"/>
                            <div className="featured-feed-info">
                                <div className="featured-feed-name">Bandcamp</div>
                                <div className="featured-feed-description">Discover new music and independent artists trending on Bandcamp.</div>
                            </div>
                        </a>
                        <a href="#" className="featured-feed-card md:col-span-4" onClick={() => switchFeed('transitSky')}>
                            <svg className="featured-feed-icon p-2" fill="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
                                <path d="M12 2C6.477 2 2 6.477 2 12s4.477 10 10 10 10-4.477 10-10S17.523 2 12 2zm-1 14H7.5c-.276 0-.5-.224-.5-.5V7.5c0-.276.224-.5.5-.5H11V16zm5.5 0H13c-.276 0-.5-.224-.5-.5V7.5c0-.276.224-.5.5-.5H16.5c.276 0 .5.224.5.5V15.5c0 .276-.224.5-.5.5z"/>
                            </svg>
                            <div className="featured-feed-info">
                                <div className="featured-feed-name">TransitSky</div>
                                <div className="featured-feed-description">Stay updated on all things public transit across the globe.</div>
                            </div>
                        </a>
                    </div>
                </div>
            </div>

            {/* Settings Modal */}
            {settingsModalOpen && (
                <div id="settingsModal" className="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50">
                    <div className="bg-white dark:bg-gray-700 p-6 rounded-lg shadow-xl w-11/12 md:w-1/3 lg:w-1/4">
                        <h3 className="text-xl font-bold mb-4 text-gray-800 dark:text-white">Settings</h3>
                        <div className="mb-4">
                            <label className="inline-flex items-center cursor-pointer">
                                <input type="checkbox" id="darkModeToggle" className="sr-only peer" checked={darkModeEnabled} onChange={(e) => setDarkModeEnabled(e.target.checked)}/>
                                <div className="relative w-11 h-6 bg-gray-200 peer-focus:outline-none peer-focus:ring-4 peer-focus:ring-blue-300 dark:peer-focus:ring-blue-800 rounded-full peer dark:bg-gray-700 peer-checked:after:translate-x-full rtl:peer-checked:after:-translate-x-full peer-checked:after:border-white after:content-[''] after:absolute after:top-[2px] after:start-[2px] after:bg-white after:border-gray-300 after:border after:rounded-full after:h-5 after:w-5 after:transition-all dark:border-gray-600 peer-checked:bg-blue-600"></div>
                                <span className="ms-3 text-sm font-medium text-gray-900 dark:text-gray-300">Dark Mode</span>
                            </label>
                        </div>
                        <div className="mb-4">
                            <label htmlFor="themeSelect" className="block text-sm font-medium text-gray-700 dark:text-gray-300 mb-2">Select Theme:</label>
                            <select id="themeSelect" className="mt-1 block w-full pl-3 pr-10 py-2 text-base border-gray-300 dark:border-gray-600 focus:outline-none focus:ring-blue-500 focus:border-blue-500 sm:text-sm rounded-md bg-gray-100 dark:bg-gray-800 text-gray-900 dark:text-gray-100" value={selectedTheme} onChange={(e) => setSelectedTheme(e.target.value)}>
                                <option value="none">None</option>
                                <option value="pride">Pride Theme</option>
                                <option value="transgenderPride">Transgender Pride Theme</option>
                                <option value="blm">BLM Theme</option>
                            </select>
                        </div>
                        <button id="closeModalBtn" className="mt-4 px-4 py-2 bg-blue-500 text-white rounded-md hover:bg-blue-600 focus:outline-none focus:ring-2 focus:ring-blue-400" onClick={() => setSettingsModalOpen(false)}>Close</button>
                    </div>
                </div>
            )}

            {/* Link Posts Modal */}
            {linkPostsModalOpen && (
                <div id="linkPostsModal" className="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50">
                    <div className="bg-white dark:bg-gray-700 p-6 rounded-lg shadow-xl w-11/12 md:w-1/2 lg:w-1/3 max-h-[90vh] flex flex-col">
                        <h3 className="text-xl font-bold mb-4 text-gray-800 dark:text-white" id="linkPostsModalTitle">{linkPostsModalContent.title}</h3>
                        <div id="linkPostsContent" className="flex-grow overflow-y-auto space-y-4">
                            {linkPostsModalContent.posts.length === 0 ? (
                                <p className="text-gray-500 text-center p-4">No recent posts found for this link.</p>
                            ) : (
                                linkPostsModalContent.posts.map((post, index) => (
                                    <div key={index} className="post-card">
                                        <div className="post-author">
                                            <div className="text-gray-700">{post.resolvedInfo.displayName}</div>
                                            <div className="text-blue-600 text-sm">@{post.resolvedInfo.handle}</div>
                                        </div>
                                        <div className="post-content">{post.textPreview}</div>
                                        <div className="post-timestamp">
                                            {new Date(post.timestamp).toLocaleString()}
                                            {post.posterTotalPosts !== undefined && ` (Posts by author: ${post.posterTotalPosts})`}
                                        </div>
                                    </div>
                                ))
                            )}
                        </div>
                        <button id="closeLinkPostsModalBtn" className="mt-6 px-4 py-2 bg-blue-500 text-white rounded-md hover:bg-blue-600 focus:outline-none focus:ring-2 focus:ring-blue-400" onClick={() => setLinkPostsModalOpen(false)}>Close</button>
                    </div>
                </div>
            )}

            {/* Plus Button Modal (New Feed Suggestion) */}
            {plusButtonModalOpen && (
                <div id="plusButtonModal" className="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50">
                    <div className="bg-white dark:bg-gray-700 p-6 rounded-lg shadow-xl w-11/12 md:w-1/2 lg:w-1/3 max-h-[90vh] flex flex-col text-center">
                        <h3 className="text-xl font-bold mb-4 text-gray-800 dark:text-white">Add a Feed to Fema's Datamaster</h3>
                        <p className="text-gray-700 dark:text-gray-300 mb-4">Want to add a Feed to Fema's datamaster? This is something I am looking to support but for now feel free to reach out to me on Bluesky <a href="https://bsky.app/profile/fema.monster" target="_blank" rel="noopener noreferrer" className="text-blue-600 hover:underline">@fema.monster</a>.</p>
                        <p className="text-gray-700 dark:text-gray-300 mb-6">Consider donating to my Ko-Fi <a href="https://ko-fi.co/urbanismplus" target="_blank" rel="noopener noreferrer" className="text-blue-600 hover:underline">(https://ko.fi/urbanismplus)</a> and mention "Datamaster" to earmark the funds for this project.</p>
                        <button id="closePlusButtonModalBtn" className="mt-4 px-4 py-2 bg-blue-500 text-white rounded-md hover:bg-blue-600 focus:outline-none focus:ring-2 focus:ring-blue-400" onClick={() => setPlusButtonModalOpen(false)}>Close</button>
                    </div>
                </div>
            )}
        </div>
    );
}

export default App;
