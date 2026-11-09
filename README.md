<!DOCTYPE html>
<html lang="en" class="dark">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>My Private Cloud Hub — Personal Cloud Orchestrator</title>
  <script src="https://cdn.tailwindcss.com/3.4.17"></script>
  <script>
    tailwind.config = {
      darkMode: 'class',
      theme: {
        extend: {
          fontFamily: {
            sans: ['Inter', 'Noto Sans Bengali', 'system-ui', '-apple-system', 'sans-serif'],
            mono: ['JetBrains Mono', 'ui-monospace', 'SFMono-Regular', 'monospace']
          }
        }
      }
    }
  </script>
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
  <link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700&family=JetBrains+Mono:wght@400;500&family=Noto+Sans+Bengali:wght@400;500;600;700&display=swap" rel="stylesheet">
  <style>
    body { font-family: 'Inter', 'Noto Sans Bengali', sans-serif; }
    .custom-scroll::-webkit-scrollbar { width: 6px; height: 6px; }
    .custom-scroll::-webkit-scrollbar-track { background: transparent; }
    .custom-scroll::-webkit-scrollbar-thumb { background: rgba(148, 163, 184, 0.3); border-radius: 9999px; }
    .fade-in { animation: fadeIn 0.2s ease-out; }
    @keyframes fadeIn { from { opacity: 0; transform: translateY(4px); } to { opacity: 1; transform: translateY(0); } }
  </style>
</head>
<body class="bg-slate-50 text-slate-900 dark:bg-[#050507] dark:text-slate-100 min-h-screen transition-colors duration-200">

  <div id="app"></div>
  <div id="modal-container"></div>
  <div id="toast-container" class="fixed bottom-5 right-5 z-50 flex flex-col gap-2 max-w-sm w-full pointer-events-none"></div>

  <script>
    // ==========================================
    // DEVICE ICON AUTO-CHANGER (5 REQUIRED TYPES)
    // ==========================================
    function getDeviceTypeIconSvg(deviceType, colorClass = "text-emerald-400") {
      const t = (deviceType || '').toLowerCase();
      if (t.includes('iphone')) {
        // iPhone Icon (Dynamic Island / Notch style)
        return `<svg class="w-6 h-6 ${colorClass}" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8"><rect x="6" y="2" width="12" height="20" rx="3"/><path d="M10 5h4"/><circle cx="12" cy="18.5" r="0.8"/></svg>`;
      }
      if (t.includes('windows') || t.includes('desktop') || t.includes('pc')) {
        // Monitor Icon
        return `<svg class="w-6 h-6 ${colorClass}" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8"><rect x="2" y="3" width="20" height="14" rx="2"/><path d="M8 21h8"/><path d="M12 17v4"/></svg>`;
      }
      if (t.includes('laptop')) {
        // Laptop Icon
        return `<svg class="w-6 h-6 ${colorClass}" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8"><rect x="3" y="4" width="18" height="12" rx="2"/><path d="M2 20h20"/></svg>`;
      }
      if (t.includes('tablet') || t.includes('ipad')) {
        // Tablet Icon
        return `<svg class="w-6 h-6 ${colorClass}" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8"><rect x="4" y="2" width="16" height="20" rx="2"/><line x1="12" y1="18" x2="12.01" y2="18"/></svg>`;
      }
      // Default: Android Phone -> Smartphone Icon
      return `<svg class="w-6 h-6 ${colorClass}" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8"><rect x="5" y="2" width="14" height="20" rx="2"/><path d="M12 18h.01"/></svg>`;
    }

    // ==========================================
    // APPLICATION STATE & STRICT ARCHITECTURE
    // ==========================================
    const PROVIDERS_CATALOG = [
      {
        id: 'cloudflare_r2',
        name: 'Cloudflare R2',
        category: 'Primary Object Storage',
        apiStatus: 'VERIFIED_S3_API',
        apiBadge: 'Official S3 API Supported',
        autoUploadCapable: true,
        quotaApiSupported: true,
        authMethod: 'S3 API Token + Private Bucket Endpoint (Server-Side Encrypted)',
        description: 'Zero-egress S3-compatible object storage. Primary Personal Storage Server for original uncompressed media.',
        color: '#F38020',
        iconSvg: `<svg class="w-6 h-6" viewBox="0 0 24 24" fill="none" stroke="#F38020" stroke-width="2"><path d="M17.5 19H9a7 7 0 1 1 6.71-9h1.79a4.5 4.5 0 1 1 0 9Z"/></svg>`
      },
      {
        id: 'google_drive',
        name: 'Google Drive',
        category: 'Cloud Drive API',
        apiStatus: 'VERIFIED_OAUTH',
        apiBadge: 'Official Google Drive API v3',
        autoUploadCapable: true,
        quotaApiSupported: true,
        authMethod: 'Separate OAuth 2.0 (drive.file narrow scope preferred)',
        description: 'Authorized file & folder browsing, explicit file uploads, and narrow-scope workspace storage.',
        color: '#3B82F6',
        iconSvg: `<svg class="w-6 h-6" viewBox="0 0 24 24" fill="none" stroke="#3B82F6" stroke-width="2"><polygon points="12 2 19 21 12 17 5 21 12 2"/></svg>`
      },
      {
        id: 'google_photos',
        name: 'Google Photos',
        category: 'Media Selection API',
        apiStatus: 'PICKER_ONLY',
        apiBadge: 'Photos Picker API Only',
        autoUploadCapable: false,
        quotaApiSupported: false,
        authMethod: 'Google Photos Picker Session API (No deprecated readonly scope)',
        description: 'User-initiated photo/video selection via official Picker API. Kept as External Cloud Library without copying into R2.',
        color: '#EC4899',
        iconSvg: `<svg class="w-6 h-6" viewBox="0 0 24 24" fill="none" stroke="#EC4899" stroke-width="2"><rect width="18" height="18" x="3" y="3" rx="2" ry="2"/><circle cx="9" cy="9" r="2"/><path d="m21 15-3.086-3.086a2 2 0 0 0-2.828 0L6 21"/></svg>`
      },
      {
        id: 'onedrive',
        name: 'Microsoft OneDrive',
        category: 'Cloud Drive API',
        apiStatus: 'VERIFIED_OAUTH',
        apiBadge: 'Microsoft Graph API',
        autoUploadCapable: true,
        quotaApiSupported: true,
        authMethod: 'OAuth 2.0 via Microsoft Identity Platform (Files.ReadWrite)',
        description: 'Personal & Microsoft 365 storage integration with chunked upload sessions and quota reporting.',
        color: '#0EA5E9',
        iconSvg: `<svg class="w-6 h-6" viewBox="0 0 24 24" fill="none" stroke="#0EA5E9" stroke-width="2"><path d="M17.5 19H9a7 7 0 1 1 6.71-9h1.79a4.5 4.5 0 1 1 0 9Z"/></svg>`
      },
      {
        id: 'dropbox',
        name: 'Dropbox',
        category: 'Cloud Drive API',
        apiStatus: 'VERIFIED_OAUTH',
        apiBadge: 'Dropbox HTTP API v2',
        autoUploadCapable: true,
        quotaApiSupported: true,
        authMethod: 'OAuth 2.0 PKCE + Offline Refresh Token (Server-Side)',
        description: 'Supports block-level upload sessions, SHA256 content hashing, and folder navigation.',
        color: '#2563EB',
        iconSvg: `<svg class="w-6 h-6" viewBox="0 0 24 24" fill="none" stroke="#2563EB" stroke-width="2"><path d="M21 16V8a2 2 0 0 0-1-1.73l-7-4a2 2 0 0 0-2 0l-7 4A2 2 0 0 0 3 8v8a2 2 0 0 0 1 1.73l7 4a2 2 0 0 0 2 0l7-4A2 2 0 0 0 21 16z"/></svg>`
      },
      {
        id: 'pcloud',
        name: 'pCloud',
        category: 'Cloud Drive API',
        apiStatus: 'VERIFIED_OAUTH',
        apiBadge: 'pCloud REST API',
        autoUploadCapable: true,
        quotaApiSupported: true,
        authMethod: 'OAuth 2.0 (US / EU Data Region Endpoint)',
        description: 'Supports direct file upload, checksum verification, and account quota inspection.',
        color: '#14B8A6',
        iconSvg: `<svg class="w-6 h-6" viewBox="0 0 24 24" fill="none" stroke="#14B8A6" stroke-width="2"><path d="M12 2v20M17 5H9.5a3.5 3.5 0 0 0 0 7h5a3.5 3.5 0 0 1 0 7H6"/></svg>`
      },
      {
        id: 'mega',
        name: 'MEGA',
        category: 'E2EE Storage',
        apiStatus: 'LIMITED_SDK',
        apiBadge: 'Client C++ SDK / Restricted Web API',
        autoUploadCapable: false,
        quotaApiSupported: false,
        authMethod: 'Pending Official Server-Side SDK Bridge Verification',
        description: 'Client-side user-controlled encryption requires Mega SDK cryptographic key derivation. Automatic routing disabled until verified.',
        color: '#EF4444',
        iconSvg: `<svg class="w-6 h-6" viewBox="0 0 24 24" fill="none" stroke="#EF4444" stroke-width="2"><circle cx="12" cy="12" r="10"/><path d="M8 15V9l4 4 4-4v6"/></svg>`
      },
      {
        id: 'terabox',
        name: 'TeraBox',
        category: 'Consumer Cloud',
        apiStatus: 'UNVERIFIED_API',
        apiBadge: 'Official Public API Unverified',
        autoUploadCapable: false,
        quotaApiSupported: false,
        authMethod: 'Unavailable — Partner OAuth Approval Required',
        description: 'Marked unavailable for automated uploads until official public developer API capabilities and terms are verified.',
        color: '#6366F1',
        iconSvg: `<svg class="w-6 h-6" viewBox="0 0 24 24" fill="none" stroke="#6366F1" stroke-width="2"><rect x="2" y="4" width="20" height="16" rx="2"/><path d="M10 4v4M14 4v4M2 12h20"/></svg>`
      },
      {
        id: 'tresorit',
        name: 'Tresorit',
        category: 'Zero-Knowledge E2EE',
        apiStatus: 'UNVERIFIED_API',
        apiBadge: 'Enterprise Zero-Knowledge API',
        autoUploadCapable: false,
        quotaApiSupported: false,
        authMethod: 'Unavailable — Zero-Knowledge Key Broker Pending',
        description: 'End-to-end encrypted storage. Automatic server-side routing disabled until official API key flow is verified.',
        color: '#10B981',
        iconSvg: `<svg class="w-6 h-6" viewBox="0 0 24 24" fill="none" stroke="#10B981" stroke-width="2"><rect width="18" height="11" x="3" y="11" rx="2" ry="2"/><path d="M7 11V7a5 5 0 0 1 10 0v4"/></svg>`
      }
    ];

    
    const LEGACY_PRESERVED_SPEC_IDS = [
      { id: 'ALL' }, { id: 'PHOTOS' }, { id: 'VIDEOS' }, { id: 'ALBUMS' }, { id: 'FOLDERS' }, { id: 'RECENT' },
      { id: 'overview' }, { id: 'devices' }, { id: 'activity' }, { id: 'settings' },
      { id: 'profile' }, { id: 'google' }, { id: 'backup' }, { id: 'allocation' }, { id: 'notifications' }, { id: 'sessions' },
      { id: 'conn_gdrive_2' }, { id: 'dev_s24_ultra' }, { id: 'dev_pixel_8' }, { id: 'dev_win_workstation' }, { id: 'dev_sim_' }
    ];

    const DEVICE_FOLDER_CATEGORIES = [
      'All Photos',
      'Camera',
      'Screenshots',
      'Downloads',
      'Pictures',
      'WhatsApp Images',
      'WhatsApp Videos',
      'Other Folders'
    ];

    const DEMO_DATASET = {
      connectedAccounts: [
        {
          id: 'conn_r2_primary',
          providerId: 'cloudflare_r2',
          label: 'Cloudflare R2 — Storage 01',
          subtitle: 'Primary Storage',
          accountIdentifier: 'r2-vault-prod (Account #8841a9)',
          status: 'HEALTHY',
          statusBadge: 'Active',
          usedGB: 650.0,
          totalGB: 1000,
          scope: 's3:PutObject, s3:GetObject, s3:ListBucket (Private Bucket)',
          connectedAt: '2026-10-14 09:15 UTC',
          lastVerified: '2 mins ago',
          priority: 1,
          thresholdPct: 85,
          enabledForUploads: true,
          role: 'PRIMARY'
        },
        {
          id: 'conn_onedrive_1',
          providerId: 'onedrive',
          label: 'Storage Provider 02 (OneDrive M365)',
          subtitle: 'Secondary Storage',
          accountIdentifier: 'ammar@example.com',
          status: 'HEALTHY',
          statusBadge: 'Active',
          usedGB: 204.8,
          totalGB: 1024,
          scope: 'Files.ReadWrite.AppFolder, offline_access',
          connectedAt: '2026-10-18 14:22 UTC',
          lastVerified: '5 mins ago',
          priority: 2,
          thresholdPct: 90,
          enabledForUploads: true,
          role: 'SECONDARY'
        },
        {
          id: 'conn_dropbox_3',
          providerId: 'dropbox',
          label: 'Storage Provider 03 (Dropbox Vault)',
          subtitle: 'Future / Overflow Storage',
          accountIdentifier: 'ammar.vault@example.com',
          status: 'HEALTHY',
          statusBadge: 'Standby',
          usedGB: 0.0,
          totalGB: 500,
          scope: 'files.content.write, files.content.read',
          connectedAt: '2026-10-20 18:40 UTC',
          lastVerified: '11 mins ago',
          priority: 3,
          thresholdPct: 90,
          enabledForUploads: true,
          role: 'ADDITIONAL'
        },
        {
          id: 'conn_gdrive_1',
          providerId: 'google_drive',
          label: 'Google Drive — Account 1 (ammar@example.com)',
          subtitle: 'External Cloud / Narrow Drive Scope',
          accountIdentifier: 'ammar@example.com',
          status: 'HEALTHY',
          statusBadge: 'External Ready',
          usedGB: 11.2,
          totalGB: 15,
          scope: 'https://www.googleapis.com/auth/drive.file',
          connectedAt: '2026-10-25 11:05 UTC',
          lastVerified: '8 mins ago',
          priority: null,
          thresholdPct: null,
          enabledForUploads: false,
          role: 'EXTERNAL_LIBRARY'
        },
        {
          id: 'conn_gphotos_1',
          providerId: 'google_photos',
          label: 'Google Photos — Picker Session (ammar@example.com)',
          subtitle: 'External Cloud Library',
          accountIdentifier: 'ammar@example.com',
          status: 'PICKER_READY',
          statusBadge: 'Picker Active',
          usedGB: null,
          totalGB: null,
          scope: 'photospicker.mediaitems.readonly (Session-Scoped)',
          connectedAt: '2026-11-01 16:30 UTC',
          lastVerified: 'Session On-Demand',
          priority: null,
          thresholdPct: null,
          enabledForUploads: false,
          role: 'IMPORT_ONLY'
        }
      ],
      devices: [
        {
          id: 'dev_01_samsung',
          code: 'Device 01',
          name: 'Samsung Galaxy',
          type: 'Android Phone',
          platformLabel: 'Android',
          onlineStatus: 'Online',
          osVersion: 'Android 15 • Companion WorkManager v1.2',
          googleEmail: 'ammar@example.com',
          deviceFingerprint: 'andr-01-s24u-9948a',
          lastActive: 'Just now',
          backedUpFiles: 1250,
          backupSizeGB: 18.4,
          lastBackupTime: 'Today, 20:41',
          backupMode: 'AUTOMATIC',
          wifiOnly: true,
          folders: ['Camera', 'Screenshots', 'WhatsApp Images', 'WhatsApp Videos', 'Downloads']
        },
        {
          id: 'dev_02_second_android',
          code: 'Device 02',
          name: 'Second Android Phone',
          type: 'Android Phone',
          platformLabel: 'Android',
          onlineStatus: 'Offline',
          osVersion: 'Android 14 • Companion WorkManager v1.2',
          googleEmail: 'second@example.com',
          deviceFingerprint: 'andr-02-px8p-3310b',
          lastActive: '3 hours ago',
          backedUpFiles: 680,
          backupSizeGB: 9.2,
          lastBackupTime: 'Today, 17:15',
          backupMode: 'AUTOMATIC',
          wifiOnly: true,
          folders: ['Camera', 'Pictures', 'Downloads']
        },
        {
          id: 'dev_03_windows',
          code: 'Device 03',
          name: 'Windows Computer',
          type: 'Windows PC',
          platformLabel: 'Desktop',
          onlineStatus: 'Online',
          osVersion: 'Windows 11 Pro 24H2 • Desktop Agent',
          googleEmail: 'ammar@example.com',
          deviceFingerprint: 'win11-03-ws01-88b4c',
          lastActive: '4 mins ago',
          backedUpFiles: 320,
          backupSizeGB: 64.7,
          lastBackupTime: 'Yesterday, 23:10',
          backupMode: 'MANUAL',
          wifiOnly: false,
          folders: ['Pictures', 'Downloads', 'Other Folders']
        },
        {
          id: 'dev_04_iphone',
          code: 'Device 04',
          name: 'iPhone 16 Pro',
          type: 'iPhone',
          platformLabel: 'iOS',
          onlineStatus: 'Online',
          osVersion: 'iOS 19 • Companion Sync',
          googleEmail: 'ammar@example.com',
          deviceFingerprint: 'ios-04-ip16p-7721d',
          lastActive: '19 mins ago',
          backedUpFiles: 410,
          backupSizeGB: 12.1,
          lastBackupTime: 'Today, 18:02',
          backupMode: 'AUTOMATIC',
          wifiOnly: true,
          folders: ['Camera', 'Screenshots']
        },
        {
          id: 'dev_05_laptop',
          code: 'Device 05',
          name: 'MacBook Pro Studio Laptop',
          type: 'Laptop',
          platformLabel: 'Laptop',
          onlineStatus: 'Offline',
          osVersion: 'macOS Sequoia • Desktop Agent',
          googleEmail: 'third.account@example.com',
          deviceFingerprint: 'mac-05-mbp16-5501e',
          lastActive: 'Yesterday',
          backedUpFiles: 195,
          backupSizeGB: 22.8,
          lastBackupTime: '2 days ago',
          backupMode: 'MANUAL',
          wifiOnly: false,
          folders: ['Pictures', 'Downloads']
        }
      ],
      mediaFiles: [
        // Samsung Galaxy (dev_01_samsung) Backed-up Files across specific folders
        {
          id: 'med_01',
          filename: 'IMG_20261109_Camera_ Sunset_RAW.DNG',
          format: 'DNG (RAW)',
          type: 'PHOTO',
          sizeMB: 42.8,
          resolution: '4000 × 3000 (12MP 16-bit)',
          captureDate: '2026-11-09 17:42:10',
          deviceId: 'dev_01_samsung',
          deviceName: 'Samsung Galaxy',
          folderCategory: 'Camera',
          folderPath: '/storage/emulated/0/DCIM/Camera/IMG_20261109_Camera_Sunset_RAW.DNG',
          providerAccountId: 'conn_r2_primary',
          providerLabel: 'Cloudflare R2 — Storage 01',
          objectKey: 'r2://pch-private-vault/dev_01_samsung/Camera/IMG_20261109_Camera_Sunset_RAW.DNG',
          checksum: 'sha256:9f86d081884c7d659a2feaa0c55ad015a3bf4f1b2b0b822cd15d6c15b0f00a08',
          status: 'Completed',
          isExternal: false,
          gradient: 'from-emerald-500/30 via-teal-600/20 to-slate-900'
        },
        {
          id: 'med_02',
          filename: 'VID_20261108_Camera_4K60_Walk.MP4',
          format: 'MP4 (HEVC 4K)',
          type: 'VIDEO',
          sizeMB: 318.5,
          resolution: '3840 × 2160 @ 60fps',
          captureDate: '2026-11-08 16:10:00',
          deviceId: 'dev_01_samsung',
          deviceName: 'Samsung Galaxy',
          folderCategory: 'Camera',
          folderPath: '/storage/emulated/0/DCIM/Camera/VID_20261108_Camera_4K60_Walk.MP4',
          providerAccountId: 'conn_r2_primary',
          providerLabel: 'Cloudflare R2 — Storage 01',
          objectKey: 'r2://pch-private-vault/dev_01_samsung/Camera/VID_20261108_Camera_4K60_Walk.MP4',
          checksum: 'sha256:4b227777d4dd1fc61c6f884f48641d02b4d121d3fd328cb08b5531fcacdabf8a',
          status: 'Completed',
          isExternal: false,
          gradient: 'from-indigo-500/30 via-blue-600/20 to-slate-900'
        },
        {
          id: 'med_03',
          filename: 'Screenshot_20261109_BankingReceipt.PNG',
          format: 'PNG (Lossless)',
          type: 'PHOTO',
          sizeMB: 3.4,
          resolution: '1440 × 3120',
          captureDate: '2026-11-09 11:20:04',
          deviceId: 'dev_01_samsung',
          deviceName: 'Samsung Galaxy',
          folderCategory: 'Screenshots',
          folderPath: '/storage/emulated/0/Pictures/Screenshots/Screenshot_20261109_BankingReceipt.PNG',
          providerAccountId: 'conn_onedrive_1',
          providerLabel: 'Storage Provider 02 (OneDrive M365)',
          objectKey: 'onedrive://root:/PersonalCloudHub/dev_01_samsung/Screenshots/Screenshot_20261109_BankingReceipt.PNG',
          checksum: 'sha256:7d865e959b2466918c9863afca942d0fb89d7c9ac0c99bafc3749504ded97730',
          status: 'Completed',
          isExternal: false,
          gradient: 'from-sky-500/30 via-cyan-600/20 to-slate-900'
        },
        {
          id: '
