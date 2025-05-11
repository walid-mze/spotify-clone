<template>
  <div class="min-h-screen bg-black text-white">
    <!-- Authentification -->
    <div v-if="!isAuthenticated" class="flex flex-col items-center justify-center min-h-screen">
      <div class="text-center p-8 max-w-md">
        <img src="/spotify-logo.svg" alt="Spotify Logo" class="w-40 mx-auto mb-8" />
        <h1 class="text-4xl font-bold mb-6">Spotify Clone</h1>
        <p class="mb-8 text-gray-300">Écoutez votre musique préférée et créez vos playlists</p>
        <button 
          @click="login" 
          class="bg-green-500 hover:bg-green-400 text-white font-bold py-3 px-6 rounded-full w-full transition duration-300"
        >
          Se connecter avec Spotify
        </button>
      </div>
    </div>

    <!-- Application principale -->
    <div v-else class="flex h-screen overflow-hidden">
      <!-- Sidebar / Navigation -->
      <div class="w-64 bg-zinc-900 flex flex-col">
        <div class="p-6">
          <img src="/spotify-logo.svg" alt="Spotify Logo" class="w-32 mb-8" />
          
          <nav class="space-y-4">
            <a href="#" @click.prevent="currentPage = 'home'" class="flex items-center space-x-3 text-gray-300 hover:text-white transition">
              <HomeIcon class="w-6 h-6" />
              <span>Accueil</span>
            </a>
            <a href="#" @click.prevent="currentPage = 'search'" class="flex items-center space-x-3 text-gray-300 hover:text-white transition">
              <SearchIcon class="w-6 h-6" />
              <span>Rechercher</span>
            </a>
            <a href="#" @click.prevent="currentPage = 'playlists'" class="flex items-center space-x-3 text-gray-300 hover:text-white transition">
              <ListMusicIcon class="w-6 h-6" />
              <span>Bibliothèque</span>
            </a>
          </nav>
        </div>
        
        <div class="mt-8 p-6">
          <h3 class="text-gray-400 uppercase text-xs font-bold mb-4">Playlists</h3>
          <div v-if="playlists.length > 0" class="space-y-2 max-h-96 overflow-y-auto">
            <div 
              v-for="playlist in playlists" 
              :key="playlist.id" 
              @click="selectPlaylist(playlist)"
              class="text-gray-300 hover:text-white cursor-pointer truncate"
            >
              {{ playlist.name }}
            </div>
          </div>
          <div v-else class="text-gray-400 text-sm">
            Aucune playlist trouvée
          </div>
          
          <button 
            @click="createPlaylist" 
            class="mt-4 flex items-center space-x-2 text-gray-300 hover:text-white"
          >
            <PlusCircleIcon class="w-5 h-5" />
            <span>Créer une playlist</span>
          </button>
        </div>
        
        <div class="mt-auto p-6">
          <div v-if="user" class="flex items-center space-x-2">
            <div class="w-8 h-8 rounded-full bg-gray-700 flex items-center justify-center">
              <UserIcon v-if="!user.images || user.images.length === 0" class="w-4 h-4 text-white" />
              <img v-else :src="user.images[0].url" class="w-8 h-8 rounded-full" alt="Profile" />
            </div>
            <span class="text-sm font-medium">{{ user.display_name }}</span>
          </div>
        </div>
      </div>
      
      <!-- Contenu principal -->
      <div class="flex-1 flex flex-col overflow-hidden">
        <!-- Header -->
        <header class="bg-zinc-800 p-4 flex items-center justify-between">
          <div class="flex items-center space-x-4">
            <button class="bg-black rounded-full p-1">
              <ChevronLeftIcon class="w-6 h-6" />
            </button>
            <button class="bg-black rounded-full p-1">
              <ChevronRightIcon class="w-6 h-6" />
            </button>
            
            <!-- Barre de recherche (visible uniquement sur la page de recherche) -->
            <div v-if="currentPage === 'search'" class="relative">
              <SearchIcon class="absolute left-3 top-1/2 transform -translate-y-1/2 text-gray-400 w-4 h-4" />
              <input 
                type="text" 
                v-model="searchQuery" 
                @input="handleSearch"
                placeholder="Que souhaitez-vous écouter ?" 
                class="bg-zinc-700 text-white pl-10 pr-4 py-2 rounded-full w-80 focus:outline-none focus:ring-2 focus:ring-white"
              />
            </div>
          </div>
          
          <div class="flex items-center space-x-3">
            <button 
              @click="logout" 
              class="bg-white text-black font-medium text-sm py-1 px-4 rounded-full hover:scale-105 transition"
            >
              Déconnexion
            </button>
          </div>
        </header>
        
        <!-- Zone de contenu -->
        <main class="flex-1 overflow-y-auto bg-gradient-to-b from-zinc-800 to-black p-8">
          <!-- Page d'accueil -->
          <div v-if="currentPage === 'home'">
            <h1 class="text-3xl font-bold mb-6">Bonjour {{ user?.display_name }}</h1>
            
            <div v-if="recentlyPlayed.length > 0">
              <h2 class="text-2xl font-bold mb-4">Écoutés récemment</h2>
              <div class="grid grid-cols-2 md:grid-cols-3 lg:grid-cols-5 gap-6">
                <div 
                  v-for="item in recentlyPlayed" 
                  :key="item.id"
                  @click="playTrack(item)"
                  class="bg-zinc-800 p-4 rounded-md hover:bg-zinc-700 transition cursor-pointer"
                >
                  <div class="relative mb-4 group">
                    <img :src="item.album.images[0].url" class="w-full aspect-square object-cover rounded-md" alt="Album cover" />
                    <button class="absolute bottom-2 right-2 bg-green-500 rounded-full p-3 shadow-lg opacity-0 group-hover:opacity-100 transition transform group-hover:translate-y-0 translate-y-2">
                      <PlayIcon class="w-5 h-5 text-black" />
                    </button>
                  </div>
                  <h3 class="font-semibold truncate">{{ item.name }}</h3>
                  <p class="text-gray-400 text-sm truncate">{{ item.artists[0].name }}</p>
                </div>
              </div>
            </div>
            
            <div class="mt-8">
              <h2 class="text-2xl font-bold mb-4">Vos playlists</h2>
              <div class="grid grid-cols-2 md:grid-cols-3 lg:grid-cols-5 gap-6">
                <div 
                  v-for="playlist in playlists.slice(0, 5)" 
                  :key="playlist.id"
                  @click="selectPlaylist(playlist)"
                  class="bg-zinc-800 p-4 rounded-md hover:bg-zinc-700 transition cursor-pointer"
                >
                  <div class="relative mb-4 group">
                    <img 
                      :src="playlist.images && playlist.images.length > 0 ? playlist.images[0].url : '/playlist-placeholder.jpg'" 
                      class="w-full aspect-square object-cover rounded-md" 
                      alt="Playlist cover" 
                    />
                    <button class="absolute bottom-2 right-2 bg-green-500 rounded-full p-3 shadow-lg opacity-0 group-hover:opacity-100 transition transform group-hover:translate-y-0 translate-y-2">
                      <PlayIcon class="w-5 h-5 text-black" />
                    </button>
                  </div>
                  <h3 class="font-semibold truncate">{{ playlist.name }}</h3>
                  <p class="text-gray-400 text-sm truncate">Playlist • {{ playlist.owner.display_name }}</p>
                </div>
              </div>
            </div>
          </div>
          
          <!-- Page de recherche -->
          <div v-else-if="currentPage === 'search'">
            <h1 class="text-3xl font-bold mb-6">Rechercher</h1>
            
            <div v-if="searchResults.length > 0">
              <h2 class="text-2xl font-bold mb-4">Résultats</h2>
              <div class="space-y-4">
                <div 
                  v-for="track in searchResults" 
                  :key="track.id"
                  @click="playTrack(track)"
                  class="flex items-center p-3 hover:bg-zinc-800 rounded-md transition cursor-pointer group"
                >
                  <div class="w-12 h-12 mr-4 relative">
                    <img :src="track.album.images[0].url" class="w-full h-full object-cover rounded" alt="Album cover" />
                    <div class="absolute inset-0 bg-black bg-opacity-50 flex items-center justify-center opacity-0 group-hover:opacity-100 transition">
                      <PlayIcon class="w-6 h-6 text-white" />
                    </div>
                  </div>
                  <div class="flex-1 min-w-0">
                    <h3 class="font-medium truncate">{{ track.name }}</h3>
                    <p class="text-gray-400 text-sm truncate">{{ track.artists.map(a => a.name).join(', ') }}</p>
                  </div>
                  <div class="text-gray-400 text-sm">{{ formatDuration(track.duration_ms) }}</div>
                  <button 
                    @click.stop="addToPlaylist(track)"
                    class="ml-4 text-gray-400 hover:text-white"
                  >
                    <PlusIcon class="w-5 h-5" />
                  </button>
                </div>
              </div>
            </div>
            
            <div v-else-if="searchQuery && !isSearching" class="text-center py-12">
              <MusicIcon class="w-16 h-16 mx-auto text-gray-500 mb-4" />
              <p class="text-gray-400">Aucun résultat trouvé pour "{{ searchQuery }}"</p>
            </div>
            
            <div v-else-if="!searchQuery" class="grid grid-cols-2 md:grid-cols-3 lg:grid-cols-4 gap-6">
              <div 
                v-for="(genre, index) in genres" 
                :key="index"
                @click="searchByGenre(genre)"
                class="aspect-square relative overflow-hidden rounded-lg cursor-pointer"
                :style="`background-color: ${getRandomColor(index)}`"
              >
                <h3 class="absolute bottom-4 left-4 text-2xl font-bold">{{ genre }}</h3>
              </div>
            </div>
          </div>
          
          <!-- Page des playlists -->
          <div v-else-if="currentPage === 'playlists'">
            <div v-if="selectedPlaylist">
              <div class="flex items-end space-x-6 mb-8">
                <img 
                  :src="selectedPlaylist.images && selectedPlaylist.images.length > 0 ? selectedPlaylist.images[0].url : '/playlist-placeholder.jpg'" 
                  class="w-56 h-56 shadow-lg" 
                  alt="Playlist cover" 
                />
                <div>
                  <p class="uppercase text-sm font-medium">Playlist</p>
                  <h1 class="text-5xl font-bold mt-2 mb-4">{{ selectedPlaylist.name }}</h1>
                  <p class="text-gray-300">{{ selectedPlaylist.description || 'Aucune description' }}</p>
                  <div class="flex items-center mt-2 text-sm text-gray-300">
                    <span class="font-medium">{{ selectedPlaylist.owner.display_name }}</span>
                    <span class="mx-1">•</span>
                    <span>{{ selectedPlaylist.tracks?.total || 0 }} titres</span>
                  </div>
                </div>
              </div>
              
              <div class="flex items-center space-x-4 mb-8">
                <button class="bg-green-500 rounded-full p-3 hover:scale-105 transition">
                  <PlayIcon class="w-6 h-6 text-black" />
                </button>
                <button @click="backToPlaylists" class="text-gray-400 hover:text-white">
                  <ArrowLeftIcon class="w-6 h-6" />
                </button>
              </div>
              
              <div class="bg-zinc-900 bg-opacity-30 rounded-md overflow-hidden">
                <div class="grid grid-cols-[16px_4fr_3fr_1fr] gap-4 px-4 py-2 border-b border-zinc-700 text-gray-400 text-sm">
                  <div>#</div>
                  <div>TITRE</div>
                  <div>ALBUM</div>
                  <div>DURÉE</div>
                </div>
                
                <div v-if="playlistTracks.length > 0" class="divide-y divide-zinc-800">
                  <div 
                    v-for="(track, index) in playlistTracks" 
                    :key="track.id"
                    @click="playTrack(track.track)"
                    class="grid grid-cols-[16px_4fr_3fr_1fr] gap-4 px-4 py-3 hover:bg-zinc-800 transition cursor-pointer items-center"
                  >
                    <div class="text-gray-400">{{ index + 1 }}</div>
                    <div class="flex items-center min-w-0">
                      <img :src="track.track.album.images[0].url" class="w-10 h-10 mr-3" alt="Album cover" />
                      <div class="min-w-0">
                        <div class="font-medium truncate">{{ track.track.name }}</div>
                        <div class="text-gray-400 text-sm truncate">{{ track.track.artists.map(a => a.name).join(', ') }}</div>
                      </div>
                    </div>
                    <div class="text-gray-400 truncate">{{ track.track.album.name }}</div>
                    <div class="text-gray-400">{{ formatDuration(track.track.duration_ms) }}</div>
                  </div>
                </div>
                
                <div v-else class="py-8 text-center text-gray-400">
                  Cette playlist ne contient aucun titre
                </div>
              </div>
            </div>
            
            <div v-else>
              <h1 class="text-3xl font-bold mb-6">Votre bibliothèque</h1>
              
              <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
                <div 
                  v-for="playlist in playlists" 
                  :key="playlist.id"
                  @click="selectPlaylist(playlist)"
                  class="flex items-center p-4 bg-zinc-800 hover:bg-zinc-700 rounded-md transition cursor-pointer"
                >
                  <img 
                    :src="playlist.images && playlist.images.length > 0 ? playlist.images[0].url : '/playlist-placeholder.jpg'" 
                    class="w-16 h-16 mr-4 object-cover" 
                    alt="Playlist cover" 
                  />
                  <div>
                    <h3 class="font-medium">{{ playlist.name }}</h3>
                    <p class="text-gray-400 text-sm">Playlist • {{ playlist.tracks?.total || 0 }} titres</p>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </main>
        
        <!-- Lecteur de musique -->
        <footer class="bg-zinc-900 border-t border-zinc-800 p-4">
          <div class="flex items-center justify-between">
            <div class="flex items-center space-x-4 w-1/3">
              <img 
                v-if="currentTrack" 
                :src="currentTrack.album?.images[0].url" 
                class="w-14 h-14 object-cover" 
                alt="Album cover" 
              />
              <div v-if="currentTrack" class="min-w-0">
                <div class="font-medium text-sm truncate">{{ currentTrack.name }}</div>
                <div class="text-gray-400 text-xs truncate">{{ currentTrack.artists?.map(a => a.name).join(', ') }}</div>
              </div>
              <div v-else class="text-gray-400 text-sm">
                Aucun titre en cours de lecture
              </div>
            </div>
            
            <div class="flex flex-col items-center w-1/3">
              <div class="flex items-center space-x-4 mb-2">
                <button class="text-gray-400 hover:text-white">
                  <ShuffleIcon class="w-5 h-5" />
                </button>
                <button class="text-gray-400 hover:text-white">
                  <SkipBackIcon class="w-5 h-5" />
                </button>
                <button 
                  @click="togglePlay" 
                  class="bg-white rounded-full p-2 hover:scale-105 transition"
                >
                  <PauseIcon v-if="isPlaying" class="w-5 h-5 text-black" />
                  <PlayIcon v-else class="w-5 h-5 text-black" />
                </button>
                <button class="text-gray-400 hover:text-white">
                  <SkipForwardIcon class="w-5 h-5" />
                </button>
                <button class="text-gray-400 hover:text-white">
                  <RepeatIcon class="w-5 h-5" />
                </button>
              </div>
              
              <div class="flex items-center space-x-2 w-full">
                <span class="text-xs text-gray-400">{{ formatDuration(currentPosition) }}</span>
                <div class="flex-1 h-1 bg-gray-700 rounded-full overflow-hidden">
                  <div 
                    class="h-full bg-gray-300 hover:bg-green-500 transition"
                    :style="`width: ${currentTrack ? (currentPosition / currentTrack.duration_ms) * 100 : 0}%`"
                  ></div>
                </div>
                <span class="text-xs text-gray-400">{{ formatDuration(currentTrack?.duration_ms || 0) }}</span>
              </div>
            </div>
            
            <div class="flex items-center justify-end space-x-3 w-1/3">
              <button class="text-gray-400 hover:text-white">
                <MicIcon class="w-4 h-4" />
              </button>
              <button class="text-gray-400 hover:text-white">
                <ListIcon class="w-4 h-4" />
              </button>
              <button class="text-gray-400 hover:text-white">
                <MonitorSpeakerIcon class="w-4 h-4" />
              </button>
              <div class="flex items-center space-x-2">
                <VolumeIcon class="w-4 h-4 text-gray-400" />
                <div class="w-24 h-1 bg-gray-700 rounded-full overflow-hidden">
                  <div class="h-full bg-gray-300 hover:bg-green-500 transition" style="width: 70%"></div>
                </div>
              </div>
              <button class="text-gray-400 hover:text-white">
                <MaximizeIcon class="w-4 h-4" />
              </button>
            </div>
          </div>
        </footer>
      </div>
    </div>
    
    <!-- Modal pour ajouter à une playlist -->
    <div v-if="showPlaylistModal" class="fixed inset-0 bg-black bg-opacity-70 flex items-center justify-center z-50">
      <div class="bg-zinc-800 rounded-md p-6 w-96 max-w-full">
        <h2 class="text-xl font-bold mb-4">Ajouter à une playlist</h2>
        <div class="max-h-80 overflow-y-auto space-y-2 mb-4">
          <div 
            v-for="playlist in playlists" 
            :key="playlist.id"
            @click="addTrackToPlaylist(playlist.id)"
            class="p-3 hover:bg-zinc-700 rounded cursor-pointer flex items-center"
          >
            <img 
              :src="playlist.images && playlist.images.length > 0 ? playlist.images[0].url : '/playlist-placeholder.jpg'" 
              class="w-10 h-10 mr-3 object-cover" 
              alt="Playlist cover" 
            />
            <span>{{ playlist.name }}</span>
          </div>
        </div>
        <div class="flex justify-end">
          <button 
            @click="showPlaylistModal = false" 
            class="bg-zinc-700 hover:bg-zinc-600 text-white px-4 py-2 rounded-md"
          >
            Annuler
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, computed, watch, onUnmounted } from 'vue';
import { 
  HomeIcon, SearchIcon, ListMusicIcon, UserIcon, ChevronLeftIcon, 
  ChevronRightIcon, PlayIcon, PauseIcon, PlusIcon, PlusCircleIcon,
  SkipBackIcon, SkipForwardIcon, ShuffleIcon, RepeatIcon, VolumeIcon,
  MicIcon, ListIcon, MonitorSpeakerIcon, MaximizeIcon, MusicIcon,
  ArrowLeftIcon
} from 'lucide-vue-next';

// État de l'authentification
const isAuthenticated = ref(false);
const accessToken = ref('');
const refreshToken = ref('');
const tokenExpiry = ref(0);

// État de l'utilisateur
const user = ref(null);
const playlists = ref([]);
const recentlyPlayed = ref([]);
const currentPage = ref('home');
const searchQuery = ref('');
const searchResults = ref([]);
const isSearching = ref(false);
const selectedPlaylist = ref(null);
const playlistTracks = ref([]);
const currentTrack = ref(null);
const isPlaying = ref(false);
const currentPosition = ref(0);
const showPlaylistModal = ref(false);
const trackToAdd = ref(null);

// Genres pour la page de recherche
const genres = [
  'Pop', 'Rock', 'Hip-Hop', 'R&B', 'Électronique', 
  'Jazz', 'Classique', 'Métal', 'Reggae', 'Country', 
  'Folk', 'Blues'
];

// Fonction pour générer une couleur aléatoire pour les genres
const getRandomColor = (index) => {
  const colors = [
    '#1DB954', '#1E3264', '#7D4698', '#E9142A', 
    '#E91429', '#BA5D07', '#056952', '#777777',
    '#509BF5', '#8D67AB', '#E8115B', '#148A08'
  ];
  return colors[index % colors.length];
};

// Fonctions d'authentification
const login = async () => {
  // Dans une vraie application, vous utiliseriez l'API Spotify OAuth
  // Pour ce TP, nous simulons l'authentification
  
  // Normalement, vous redirigeriez vers:
  // const clientId = 'YOUR_CLIENT_ID';
  // const redirectUri = encodeURIComponent(window.location.origin);
  // const scope = encodeURIComponent('user-read-private user-read-email playlist-read-private playlist-modify-private playlist-modify-public user-read-recently-played');
  // window.location.href = `https://accounts.spotify.com/authorize?client_id=${clientId}&response_type=code&redirect_uri=${redirectUri}&scope=${scope}`;
  
  // Simulation pour le TP
  isAuthenticated.value = true;
  accessToken.value = 'simulated-token';
  
  // Charger les données simulées
  await loadUserData();
  await loadPlaylists();
  await loadRecentlyPlayed();
};

const logout = () => {
  isAuthenticated.value = false;
  accessToken.value = '';
  refreshToken.value = '';
  user.value = null;
  playlists.value = [];
  recentlyPlayed.value = [];
  currentTrack.value = null;
  isPlaying.value = false;
};

// Chargement des données utilisateur
const loadUserData = async () => {
  // Simulation de l'appel API
  // En réalité: const response = await fetch('https://api.spotify.com/v1/me', { headers: { 'Authorization': `Bearer ${accessToken.value}` } });
  
  // Données simulées
  user.value = {
    display_name: 'Utilisateur Spotify',
    id: 'user123',
    images: []
  };
};

const loadPlaylists = async () => {
  // Simulation de l'appel API
  // En réalité: const response = await fetch('https://api.spotify.com/v1/me/playlists', { headers: { 'Authorization': `Bearer ${accessToken.value}` } });
  
  // Données simulées
  playlists.value = [
    {
      id: 'playlist1',
      name: 'Mes favoris',
      description: 'Mes titres préférés',
      owner: { display_name: user.value.display_name },
      images: [{ url: 'https://i.scdn.co/image/ab67706c0000da84fcb8b92f2615d3261b8eb146' }],
      tracks: { total: 25 }
    },
    {
      id: 'playlist2',
      name: 'Découvertes de la semaine',
      description: 'Nouveaux titres recommandés',
      owner: { display_name: 'Spotify' },
      images: [{ url: 'https://i.scdn.co/image/ab67706f00000002fe24d7084be472288cd6ee6c' }],
      tracks: { total: 30 }
    },
    {
      id: 'playlist3',
      name: 'Ambiance chill',
      description: 'Pour se détendre',
      owner: { display_name: user.value.display_name },
      images: [{ url: 'https://i.scdn.co/image/ab67706c0000da8477a1b7da75eaae12e658768f' }],
      tracks: { total: 18 }
    }
  ];
};

const loadRecentlyPlayed = async () => {
  // Simulation de l'appel API
  // En réalité: const response = await fetch('https://api.spotify.com/v1/me/player/recently-played', { headers: { 'Authorization': `Bearer ${accessToken.value}` } });
  
  // Données simulées
  recentlyPlayed.value = [
    {
      id: 'track1',
      name: 'Blinding Lights',
      artists: [{ name: 'The Weeknd' }],
      album: { 
        name: 'After Hours',
        images: [{ url: 'https://i.scdn.co/image/ab67616d0000b273ef12a4e9d43ebe7f17b2af5e' }]
      },
      duration_ms: 200000
    },
    {
      id: 'track2',
      name: 'Dance Monkey',
      artists: [{ name: 'Tones and I' }],
      album: { 
        name: 'The Kids Are Coming',
        images: [{ url: 'https://i.scdn.co/image/ab67616d0000b273b72d78996387eadb6c0f5884' }]
      },
      duration_ms: 210000
    },
    {
      id: 'track3',
      name: 'Watermelon Sugar',
      artists: [{ name: 'Harry Styles' }],
      album: { 
        name: 'Fine Line',
        images: [{ url: 'https://i.scdn.co/image/ab67616d0000b2739e495fb707973f3390850eea' }]
      },
      duration_ms: 174000
    },
    {
      id: 'track4',
      name: 'Don\'t Start Now',
      artists: [{ name: 'Dua Lipa' }],
      album: { 
        name: 'Future Nostalgia',
        images: [{ url: 'https://i.scdn.co/image/ab67616d0000b273bd26ede1ae69327010d49946' }]
      },
      duration_ms: 183000
    },
    {
      id: 'track5',
      name: 'Circles',
      artists: [{ name: 'Post Malone' }],
      album: { 
        name: 'Hollywood\'s Bleeding',
        images: [{ url: 'https://i.scdn.co/image/ab67616d0000b273b1c4b76e23414c9f20242268' }]
      },
      duration_ms: 215000
    }
  ];
};

// Fonctions de recherche
const handleSearch = () => {
  if (!searchQuery.value) {
    searchResults.value = [];
    return;
  }
  
  isSearching.value = true;
  
  // Simulation d'un délai de recherche
  setTimeout(() => {
    // Simulation de l'appel API
    // En réalité: const response = await fetch(`https://api.spotify.com/v1/search?q=${encodeURIComponent(searchQuery.value)}&type=track`, { headers: { 'Authorization': `Bearer ${accessToken.value}` } });
    
    // Données simulées
    searchResults.value = [
      {
        id: 'search1',
        name: `${searchQuery.value} - Hit Song`,
        artists: [{ name: 'Popular Artist' }],
        album: { 
          name: 'Greatest Hits',
          images: [{ url: 'https://i.scdn.co/image/ab67616d0000b273ba5db46f4b838ef6027e6f96' }]
        },
        duration_ms: 180000
      },
      {
        id: 'search2',
        name: `Remix of ${searchQuery.value}`,
        artists: [{ name: 'DJ Remix' }],
        album: { 
          name: 'Remix Collection',
          images: [{ url: 'https://i.scdn.co/image/ab67616d0000b2732fbd77033247e889cb7d2ac4' }]
        },
        duration_ms: 195000
      },
      {
        id: 'search3',
        name: `${searchQuery.value} Acoustic Version`,
        artists: [{ name: 'Acoustic Band' }],
        album: { 
          name: 'Unplugged',
          images: [{ url: 'https://i.scdn.co/image/ab67616d0000b273e719a8e4b1e694bc3f970679' }]
        },
        duration_ms: 210000
      }
    ];
    
    isSearching.value = false;
  }, 500);
};

const searchByGenre = (genre) => {
  searchQuery.value = genre;
  handleSearch();
};

// Fonctions de gestion des playlists
const selectPlaylist = async (playlist) => {
  selectedPlaylist.value = playlist;
  currentPage.value = 'playlists';
  
  // Simulation de l'appel API
  // En réalité: const response = await fetch(`https://api.spotify.com/v1/playlists/${playlist.id}/tracks`, { headers: { 'Authorization': `Bearer ${accessToken.value}` } });
  
  // Données simulées
  playlistTracks.value = Array(10).fill().map((_, i) => ({
    added_at: new Date().toISOString(),
    track: {
      id: `playlist-track-${i}`,
      name: `Titre ${i + 1} de la playlist`,
      artists: [{ name: 'Artiste de la playlist' }],
      album: { 
        name: `Album ${i % 3 + 1}`,
        images: [{ url: `https://i.scdn.co/image/ab67616d0000b273${Math.random().toString(36).substring(2, 8)}` }]
      },
      duration_ms: 180000 + i * 10000
    }
  }));
};

const backToPlaylists = () => {
  selectedPlaylist.value = null;
};

const createPlaylist = async () => {
  // Simulation de l'appel API
  // En réalité: const response = await fetch('https://api.spotify.com/v1/users/me/playlists', { method: 'POST', headers: { 'Authorization': `Bearer ${accessToken.value}` }, body: JSON.stringify({ name: 'Nouvelle playlist', description: 'Créée depuis mon clone Spotify', public: false }) });
  
  // Ajouter la nouvelle playlist à la liste
  const newPlaylist = {
    id: `playlist-${Date.now()}`,
    name: 'Nouvelle playlist',
    description: 'Créée depuis mon clone Spotify',
    owner: { display_name: user.value.display_name },
    images: [],
    tracks: { total: 0 }
  };
  
  playlists.value.unshift(newPlaylist);
  selectPlaylist(newPlaylist);
};

// Fonctions du lecteur de musique
const playTrack = (track) => {
  currentTrack.value = track;
  isPlaying.value = true;
  currentPosition.value = 0;
  
  // Simulation de la progression de la lecture
  startPlaybackSimulation();
};

const togglePlay = () => {
  isPlaying.value = !isPlaying.value;
  
  if (isPlaying.value) {
    startPlaybackSimulation();
  }
};

let playbackInterval;

const startPlaybackSimulation = () => {
  clearInterval(playbackInterval);
  
  if (!currentTrack.value) return;
  
  playbackInterval = setInterval(() => {
    if (isPlaying.value) {
      currentPosition.value += 1000; // Avance d'une seconde
      
      if (currentPosition.value >= currentTrack.value.duration_ms) {
        // Fin de la piste
        currentPosition.value = 0;
        isPlaying.value = false;
        clearInterval(playbackInterval);
      }
    }
  }, 1000);
};

// Fonction pour ajouter un morceau à une playlist
const addToPlaylist = (track) => {
  trackToAdd.value = track;
  showPlaylistModal.value = true;
};

const addTrackToPlaylist = async (playlistId) => {
  // Simulation de l'appel API
  // En réalité: const response = await fetch(`https://api.spotify.com/v1/playlists/${playlistId}/tracks`, { method: 'POST', headers: { 'Authorization': `Bearer ${accessToken.value}` }, body: JSON.stringify({ uris: [`spotify:track:${trackToAdd.value.id}`] }) });
  
  // Mise à jour de l'interface
  const playlistIndex = playlists.value.findIndex(p => p.id === playlistId);
  if (playlistIndex !== -1) {
    playlists.value[playlistIndex].tracks.total += 1;
  }
  
  showPlaylistModal.value = false;
  trackToAdd.value = null;
};

// Utilitaires
const formatDuration = (ms) => {
  const minutes = Math.floor(ms / 60000);
  const seconds = Math.floor((ms % 60000) / 1000);
  return `${minutes}:${seconds.toString().padStart(2, '0')}`;
};

// Initialisation
onMounted(() => {
  // Vérifier si un token est déjà stocké
  const storedToken = localStorage.getItem('spotify_access_token');
  if (storedToken) {
    accessToken.value = storedToken;
    isAuthenticated.value = true;
    
    // Charger les données
    loadUserData();
    loadPlaylists();
    loadRecentlyPlayed();
  }
});

// Nettoyage
onUnmounted(() => {
  clearInterval(playbackInterval);
});
</script>

<style>
/* Styles globaux */
body {
  font-family: 'Montserrat', sans-serif;
  margin: 0;
  padding: 0;
}

/* Personnalisation de la barre de défilement */
::-webkit-scrollbar {
  width: 12px;
}

::-webkit-scrollbar-track {
  background: #121212;
}

::-webkit-scrollbar-thumb {
  background: #535353;
  border-radius: 6px;
}

::-webkit-scrollbar-thumb:hover {
  background: #7d7d7d;
}
</style>