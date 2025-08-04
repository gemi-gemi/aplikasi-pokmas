import React, { useState, useEffect, useRef } from 'react';
import { Home, Users, DollarSign, Briefcase, FileText, ClipboardList, HandCoins, ShieldCheck, Plus, Search, MoreVertical, X, Calendar, ChevronDown, Edit, Trash2, ShoppingCart, Download, LogOut, Upload, CheckCircle, Paperclip, DownloadCloud, Replace, XCircle, Building, RefreshCw, Sparkles, HelpCircle } from 'lucide-react';

// --- KONEKSI FIREBASE ---
import { initializeApp } from 'firebase/app';
import { getAuth, signInAnonymously } from 'firebase/auth';
import { getFirestore, collection, onSnapshot, addDoc, updateDoc, deleteDoc, doc, writeBatch } from 'firebase/firestore';
import { getStorage, ref, uploadBytes, getDownloadURL, deleteObject } from "firebase/storage";

// --- KONFIGURASI GOOGLE SHEETS ---
const SCRIPT_URL = 'https://script.google.com/macros/s/AKfycbwXQ09RI31tBSm735pbEC_xtcXfUnSpiFK4hdgt4qEkMYOAYOQD1QnNJ2FiTtU5-9iGaw/exec';

// Konfigurasi Firebase Anda telah dimasukkan.
const firebaseConfig = {
  apiKey: "AIzaSyBqJ6u8VAmTINkDF3C92IPLtGO0Rb1IA6U",
  authDomain: "pokmas-karya-25.firebaseapp.com",
  projectId: "pokmas-karya-25",
  storageBucket: "pokmas-karya-25.appspot.com",
  messagingSenderId: "643831785993",
  appId: "1:643831785993:web:d3ca3b5bfe10879be566d8",
  measurementId: "G-8ZXK6TR418"
};


// Inisialisasi Firebase, Firestore, Auth, dan Storage
const app = initializeApp(firebaseConfig);
const db = getFirestore(app);
const auth = getAuth(app);
const storage = getStorage(app);

// --- FUNGSI PANGGILAN GEMINI API ---
const callGeminiApi = async (prompt) => {
  const apiKey = ""; // Disediakan oleh environment saat runtime
  const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-05-20:generateContent?key=${apiKey}`;

  const payload = {
    contents: [{
      parts: [{ text: prompt }]
    }]
  };

  // Implementasi exponential backoff untuk percobaan ulang
  let response;
  let retries = 3;
  let delay = 1000;
  while (retries > 0) {
    try {
      response = await fetch(apiUrl, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(payload)
      });

      if (response.ok) {
        const result = await response.json();
        if (result.candidates && result.candidates.length > 0 &&
            result.candidates[0].content && result.candidates[0].content.parts &&
            result.candidates[0].content.parts.length > 0) {
          return result.candidates[0].content.parts[0].text;
        } else {
          throw new Error("Struktur respons API tidak valid.");
        }
      } else {
         throw new Error(`API request failed with status ${response.status}`);
      }
    } catch (error) {
      console.error(`API call failed: ${error.message}. Retrying in ${delay / 1000}s...`);
      retries--;
      if (retries === 0) {
        throw error; 
      }
      await new Promise(res => setTimeout(res, delay));
      delay *= 2;
    }
  }
  return "Gagal menghasilkan teks setelah beberapa kali percobaan.";
};

// --- FUNGSI UNTUK MENGIRIM DATA KE GOOGLE SHEETS ---
const sendDataToGoogleSheet = async (payload) => {
    if (SCRIPT_URL.includes('PASTE_YOUR')) {
        console.warn("Google Apps Script URL belum dikonfigurasi. Sinkronisasi data dilewati.");
        return;
    }
    try {
        fetch(SCRIPT_URL, {
            method: 'POST',
            headers: { "Content-Type": "text/plain;charset=utf-8" },
            body: JSON.stringify(payload),
        });
    } catch (error) {
        console.error('Gagal mengirim data ke Google Sheet:', error);
    }
};

// --- KOMPONEN BANTUAN ---
const FirebaseErrorDisplay = ({ message, onRetry }) => (
    <div className="flex items-center justify-center h-screen bg-slate-100">
        <div className="w-full max-w-lg p-8 text-center bg-white rounded-2xl shadow-lg">
            <XCircle className="w-16 h-16 mx-auto text-red-500" />
            <h2 className="mt-4 text-2xl font-bold text-slate-800">Koneksi Gagal</h2>
            <p className="mt-2 text-slate-600">Terjadi masalah saat menghubungkan ke database:</p>
            <div className="mt-4 p-3 text-sm text-left text-red-800 bg-red-100 rounded-lg">
                <p className="font-mono">{message}</p>
            </div>
            <p className="mt-4 text-sm text-slate-500">
                <b>Solusi:</b> Masalah ini biasanya terjadi karena <b>Aturan Keamanan (Security Rules)</b> di Firebase Console belum diatur. Pastikan aturan Firestore dan Storage mengizinkan akses baca/tulis untuk pengguna yang terotentikasi.
            </p>
            <button onClick={onRetry} className="mt-6 px-4 py-2 text-sm font-medium text-white bg-indigo-600 rounded-lg hover:bg-indigo-700">
                Coba Lagi
            </button>
        </div>
    </div>
);


// --- KOMPONEN LOGIN ---
const LoginPage = ({ onLogin }) => {
    const [username, setUsername] = useState('');
    const [password, setPassword] = useState('');
    const [error, setError] = useState('');

    const handleSubmit = (e) => {
        e.preventDefault();
        if ((username === 'ketua' && password === 'karya25') || (username === 'staff' && password === 'karya25')) {
            onLogin({ username });
        } else {
            setError('Username atau password salah.');
        }
    };

    return (
        <div className="flex items-center justify-center min-h-screen bg-slate-100">
            <div className="w-full max-w-md p-8 space-y-8 bg-white rounded-2xl shadow-lg">
                <div className="text-center">
                    <div className="flex justify-center mb-4">
                        <div className="p-3 bg-indigo-100 rounded-full">
                           <Building className="w-8 h-8 text-indigo-600" />
                        </div>
                    </div>
                    <h2 className="text-2xl font-bold tracking-tight text-slate-800">
                        POKMAS KARYA 25
                    </h2>
                    <p className="mt-2 text-sm text-slate-500">Sistem Keuangan & Administrasi</p>
                </div>
                <form className="mt-8 space-y-6" onSubmit={handleSubmit}>
                    <div className="space-y-4">
                        <div>
                            <input id="username" name="username" type="text" required className="relative block w-full px-4 py-3 text-slate-900 placeholder-slate-500 bg-slate-50 border border-slate-300 rounded-lg appearance-none focus:outline-none focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500 sm:text-sm" placeholder="Username" value={username} onChange={(e) => setUsername(e.target.value)} />
                        </div>
                        <div>
                            <input id="password" name="password" type="password" required className="relative block w-full px-4 py-3 text-slate-900 placeholder-slate-500 bg-slate-50 border border-slate-300 rounded-lg appearance-none focus:outline-none focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500 sm:text-sm" placeholder="Password" value={password} onChange={(e) => setPassword(e.target.value)} />
                        </div>
                    </div>
                    {error && <p className="text-sm text-center text-red-600">{error}</p>}
                    <div>
                        <button type="submit" className="relative flex justify-center w-full px-4 py-3 text-sm font-medium text-white bg-indigo-600 border border-transparent rounded-lg shadow-sm group hover:bg-indigo-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500 transition-all duration-300">
                            Login
                        </button>
                    </div>
                </form>
            </div>
        </div>
    );
};


// --- KOMPONEN UTAMA ---
const App = () => {
    const [user, setUser] = useState(null);
    const [isFirebaseReady, setIsFirebaseReady] = useState(false);
    const [firebaseError, setFirebaseError] = useState(null);
    const [currentPage, setCurrentPage] = useState('Dashboard');
    const [isModalOpen, setIsModalOpen] = useState(false);
    const [ModalComponent, setModalComponent] = useState(null);

    const [members, setMembers] = useState([]);
    const [workPackages, setWorkPackages] = useState([]);
    const [transactions, setTransactions] = useState([]);
    const [wageBills, setWageBills] = useState([]);
    const [materialBills, setMaterialBills] = useState([]);
    const [loading, setLoading] = useState(true);

    useEffect(() => {
        const authenticate = async () => {
            try {
                await signInAnonymously(auth);
                setIsFirebaseReady(true);
            } catch (error) {
                console.error("Gagal melakukan otentikasi anonim Firebase:", error);
                if (error.code === 'auth/network-request-failed') {
                    setFirebaseError("Koneksi ke server otentikasi gagal. Pastikan Anda terhubung ke internet dan Anonymous Sign-in diaktifkan di Firebase Console (Authentication > Sign-in method).");
                } else {
                    setFirebaseError("Gagal terhubung ke server. Periksa konsol untuk detailnya.");
                }
            }
        };
        authenticate();
    }, []);

    useEffect(() => {
        if (!user || !isFirebaseReady) return;
        
        setLoading(true);
        setFirebaseError(null);
        const collections = {
            members: setMembers, "work-packages": setWorkPackages,
            transactions: setTransactions, "wage-bills": setWageBills,
            "material-bills": setMaterialBills,
        };

        const unsubs = Object.entries(collections).map(([name, setter]) => 
            onSnapshot(collection(db, name), (snapshot) => {
                setter(snapshot.docs.map(doc => ({ id: doc.id, ...doc.data() })));
            }, (error) => {
                console.error(`Error pada snapshot listener untuk ${name}:`, error);
                setFirebaseError("Gagal mengambil data: Izin ditolak. Pastikan Aturan Keamanan (Security Rules) di Firestore sudah diatur untuk mengizinkan akses baca (read).");
            })
        );
        
        setLoading(false);
        return () => {
            unsubs.forEach(unsub => unsub());
        }
    }, [user, isFirebaseReady]);

    const totalPemasukan = transactions.filter(t => t.jenis === 'Pemasukan' && t.status === 'Cair').reduce((sum, t) => sum + t.jumlah, 0);
    const totalPengeluaranUmum = transactions.filter(t => t.jenis === 'Pengeluaran').reduce((sum, t) => sum + t.jumlah, 0);
    const totalUpah = wageBills.reduce((sum, bill) => sum + bill.jumlah, 0);
    const totalMaterial = materialBills.reduce((sum, bill) => sum + bill.total, 0);
    const totalPengeluaran = totalPengeluaranUmum + totalUpah + totalMaterial;
    const saldoTotal = totalPemasukan - totalPengeluaran;
    const totalAnggota = members.length;
    const totalPaketPekerjaan = workPackages.length;

    const openModal = (Component, props = {}) => {
        setModalComponent(() => () => <Component closeModal={closeModal} {...props} />);
        setIsModalOpen(true);
    };

    const closeModal = () => {
        setIsModalOpen(false);
        setModalComponent(null);
    };

    const handleAddOrUpdate = async (type, data, id = null, file = null) => {
        const collectionName = {
            anggota: 'members', transaksi: 'transactions', paket: 'work-packages',
            'tagihan-upah': 'wage-bills', 'tagihan-material': 'material-bills'
        }[type];

        try {
            if (id) {
                const docRef = doc(db, collectionName, id);
                await updateDoc(docRef, data);
                if (file) {
                    await handleFileUpload(id, file);
                }
            } else {
                const docRef = await addDoc(collection(db, collectionName), data);
                if (file) {
                    await handleFileUpload(docRef.id, file);
                }
            }
        } catch (error) {
            console.error("Error saat menyimpan data:", error);
            alert("Gagal menyimpan data. Periksa koneksi dan aturan keamanan Firebase.");
        }
        closeModal();
    };
    
    const handleFileUpload = async (packageId, file) => {
        if (!file) return;
        const storageRef = ref(storage, `attachments/${packageId}/${file.name}`);
        
        try {
            await uploadBytes(storageRef, file);
            const downloadURL = await getDownloadURL(storageRef);
            
            const docRef = doc(db, 'work-packages', packageId);
            await updateDoc(docRef, {
                fileName: file.name,
                fileURL: downloadURL,
            });
        } catch (error) {
            console.error("Error saat mengunggah file:", error);
            alert("Gagal mengunggah file. Pastikan aturan keamanan Firebase Storage sudah benar.");
        }
    };

    const handleFileDelete = async (packageId, fileName) => {
        const storageRef = ref(storage, `attachments/${packageId}/${fileName}`);

        try {
            await deleteObject(storageRef);
            const docRef = doc(db, 'work-packages', packageId);
            await updateDoc(docRef, {
                fileName: null,
                fileURL: null,
            });
            closeModal();
        } catch (error) {
            console.error("Error saat menghapus file:", error);
            alert("Gagal menghapus file. Periksa koneksi dan aturan keamanan Firebase Storage.");
        }
    };
    
    const handleBulkAdd = async (type, dataArray) => {
        const collectionName = {
            transaksi: 'transactions',
            paket: 'work-packages',
        }[type];
    
        const batch = writeBatch(db);
    
        dataArray.forEach((item) => {
            const processedItem = { ...item };
            if (type === 'paket' && processedItem.nilaiPagu) {
                processedItem.nilaiPagu = Number(processedItem.nilaiPagu);
            }
    
            const docRef = doc(collection(db, collectionName));
            batch.set(docRef, processedItem);
        });
    
        await batch.commit();
        closeModal();
    };

    const handleDisburseFromPackagePage = async (workPackage, stageName) => {
        const stages = { 'Tahap 1': 0.4, 'Tahap 2': 0.3, 'Tahap 3': 0.3 };
        const percentage = stages[stageName];
        if (!percentage) return;

        const transactionData = {
            tanggal: new Date().toISOString().split('T')[0],
            jenis: 'Pemasukan',
            kategori: `Pencairan Dana ${stageName}`,
            jumlah: Number(workPackage.nilaiPagu) * percentage,
            paketPekerjaan: workPackage.namaPaket,
            keterangan: `Dana dicairkan untuk ${stageName} (${percentage * 100}%) paket: ${workPackage.namaPaket}`,
            status: 'Cair',
        };
        await addDoc(collection(db, 'transactions'), transactionData);
    };

    const handleDelete = async (ids, type) => {
        const collectionName = {
            anggota: 'members', transaksi: 'transactions', paket: 'work-packages',
            'tagihan-upah': 'wage-bills', 'tagihan-material': 'material-bills'
        }[type];
        const batch = writeBatch(db);
        ids.forEach(id => {
            const docRef = doc(db, collectionName, id);
            batch.delete(docRef);
        });
        await batch.commit();
        closeModal();
    };
    
    const handleResetWorkPackages = async () => {
        const batch = writeBatch(db);

        // Hapus file dari Storage
        for (const wp of workPackages) {
            if (wp.fileName) {
                const storageRef = ref(storage, `attachments/${wp.id}/${wp.fileName}`);
                try {
                    await deleteObject(storageRef);
                } catch (error) {
                    console.error(`Gagal menghapus file ${wp.fileName}:`, error);
                }
            }
            
            // Hapus transaksi terkait
            const relatedTransactions = transactions.filter(t => t.paketPekerjaan === wp.namaPaket);
            relatedTransactions.forEach(t => {
                const txDocRef = doc(db, 'transactions', t.id);
                batch.delete(txDocRef);
            });

            // Hapus paket pekerjaan
            const wpDocRef = doc(db, 'work-packages', wp.id);
            batch.delete(wpDocRef);
        }

        await batch.commit();
        closeModal();
    };

    const confirmDelete = (ids, type) => {
        const validIds = ids.filter(id => id);
        if (validIds.length === 0) {
            console.warn("Delete called with no valid IDs.");
            return;
        }
        const itemText = {
            anggota: 'anggota', transaksi: 'transaksi', paket: 'paket pekerjaan',
            'tagihan-upah': 'tagihan upah', 'tagihan-material': 'tagihan material'
        }[type];
        openModal(ConfirmationModal, {
            onConfirm: () => handleDelete(validIds, type),
            title: `Hapus Data`,
            message: `Apakah Anda yakin ingin menghapus ${validIds.length} data ${itemText} yang dipilih? Tindakan ini tidak dapat dibatalkan.`
        });
    };
    
    const confirmResetWorkPackages = () => {
        openModal(ConfirmationModal, {
            onConfirm: handleResetWorkPackages,
            title: 'Reset Semua Paket Pekerjaan?',
            message: 'Tindakan ini akan menghapus SEMUA data paket pekerjaan, SEMUA transaksi pencairan dana terkait, dan SEMUA file lampiran yang terhubung. Data yang sudah dihapus tidak dapat dikembalikan. Apakah Anda yakin?'
        });
    };

    const handleLogin = (userData) => { setUser(userData); };
    const handleLogout = () => { setUser(null); };

    if (firebaseError) {
        return <FirebaseErrorDisplay message={firebaseError} onRetry={() => window.location.reload()} />;
    }
    
    if (!user) { return <LoginPage onLogin={handleLogin} />; }

    const renderPage = () => {
        if (loading || !isFirebaseReady) { return <div className="flex justify-center items-center h-64">Menyiapkan koneksi...</div> }
        switch (currentPage) {
            case 'Dashboard': return <DashboardPage stats={{ saldoTotal, totalAnggota, totalPaketPekerjaan, totalPemasukan, totalPengeluaran, totalUpah, totalMaterial }} />;
            case 'Anggota': return <AnggotaPage members={members} openModal={openModal} confirmDelete={confirmDelete} handleAddOrUpdate={handleAddOrUpdate} />;
            case 'Keuangan': return <KeuanganPage transactions={transactions} workPackages={workPackages} openModal={openModal} confirmDelete={confirmDelete} handleAddOrUpdate={handleAddOrUpdate} />;
            case 'Paket Pekerjaan': return <PaketPekerjaanPage workPackages={workPackages} wageBills={wageBills} transactions={transactions} openModal={openModal} confirmDelete={confirmDelete} handleAddOrUpdate={handleAddOrUpdate} handleFileDelete={handleFileDelete} handleFileUpload={handleFileUpload} handleBulkAdd={handleBulkAdd} confirmReset={confirmResetWorkPackages} handleDisburseFromPackagePage={handleDisburseFromPackagePage} />;
            case 'Tagihan Upah': return <TagihanUpahPage wageBills={wageBills} openModal={openModal} workPackages={workPackages} confirmDelete={confirmDelete} handleAddOrUpdate={handleAddOrUpdate} />;
            case 'Tagihan Material': return <TagihanMaterialPage materialBills={materialBills} openModal={openModal} workPackages={workPackages} confirmDelete={confirmDelete} handleAddOrUpdate={handleAddOrUpdate} />;
            case 'Laporan': return <LaporanPage data={{members, workPackages, transactions, wageBills, materialBills}} stats={{ totalPemasukan, totalPengeluaran, saldoTotal, totalUpah, totalMaterial }} />;
            case 'Tanya RAB': return <TanyaRabPage />;
            default: return <DashboardPage stats={{ saldoTotal, totalAnggota, totalPaketPekerjaan, totalPemasukan, totalPengeluaran, totalUpah, totalMaterial }} />;
        }
    };
    
    const navItems = [
        { name: 'Dashboard', icon: Home, page: 'Dashboard' },
        { name: 'Anggota', icon: Users, page: 'Anggota' },
        { name: 'Keuangan', icon: DollarSign, page: 'Keuangan' },
        { name: 'Paket Pekerjaan', icon: Briefcase, page: 'Paket Pekerjaan' },
        { name: 'Tagihan Upah', icon: ClipboardList, page: 'Tagihan Upah' },
        { name: 'Tagihan Material', icon: ShoppingCart, page: 'Tagihan Material' },
        { name: 'Laporan', icon: FileText, page: 'Laporan' },
        { name: 'Tanya RAB', icon: HelpCircle, page: 'Tanya RAB' },
    ];

    return (
        <div className="flex h-screen bg-slate-100 font-sans">
            <aside className="w-64 bg-white border-r border-slate-200 flex-shrink-0">
                <div className="p-6 flex items-center space-x-3">
                    <div className="p-2 bg-indigo-100 rounded-lg">
                        <Building className="w-6 h-6 text-indigo-600" />
                    </div>
                    <h1 className="text-lg font-bold text-slate-800">POKMAS KARYA 25</h1>
                </div>
                <nav className="mt-4 px-4">
                    {navItems.map(item => (
                        <div key={item.name}>
                            <a href="#" 
                               onClick={(e) => {
                                   e.preventDefault();
                                   setCurrentPage(item.page);
                               }} 
                               className={`flex items-center justify-between py-2.5 px-4 rounded-lg text-slate-600 transition-all duration-200 ${currentPage === item.page ? 'bg-indigo-50 text-indigo-600 font-semibold' : 'hover:bg-slate-100'}`}>
                                <div className="flex items-center">
                                    <item.icon className="w-5 h-5" />
                                    <span className="ml-3">{item.name}</span>
                                </div>
                            </a>
                        </div>
                    ))}
                </nav>
            </aside>
            <main className="flex-1 overflow-y-auto">
                <header className="bg-white/80 backdrop-blur-lg sticky top-0 z-10 border-b border-slate-200 p-4 flex justify-between items-center">
                    <div></div>
                    <div className="flex items-center">
                        <span className="text-slate-700 font-medium mr-4">Login sebagai: <strong className="capitalize">{user.username}</strong></span>
                        <button onClick={handleLogout} className="flex items-center text-sm text-red-600 hover:text-red-800">
                            <LogOut className="w-4 h-4 mr-1" />
                            Logout
                        </button>
                    </div>
                </header>
                <div className="p-6 md:p-8">{renderPage()}</div>
            </main>
            {isModalOpen && ModalComponent && (
                <div className="fixed inset-0 bg-black bg-opacity-60 flex items-center justify-center z-50 p-4 animate-fade-in">
                    <div className="bg-white rounded-xl shadow-2xl w-full max-w-lg transform transition-all animate-scale-in"><ModalComponent /></div>
                </div>
            )}
        </div>
    );
};

// --- KOMPONEN HALAMAN ---
const DashboardPage = ({ stats }) => {
    const formatRupiah = (number) => new Intl.NumberFormat('id-ID', { style: 'currency', currency: 'IDR', minimumFractionDigits: 0 }).format(number);
    const statCards = [
        { title: 'Saldo Total', value: formatRupiah(stats.saldoTotal), icon: DollarSign, color: 'green' },
        { title: 'Total Anggota', value: stats.totalAnggota, icon: Users, color: 'blue' },
        { title: 'Paket Pekerjaan', value: stats.totalPaketPekerjaan, icon: Briefcase, color: 'purple' },
        { title: 'Total Pemasukan', value: formatRupiah(stats.totalPemasukan), icon: Plus, color: 'teal' },
    ];
    
    const totalOperasional = stats.totalUpah + stats.totalMaterial;

    return (
        <div>
            <h2 className="text-3xl font-bold text-slate-800">Dashboard</h2>
            <p className="text-slate-500 mt-1">Ringkasan sistem pembukuan POKMAS</p>
            <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-6 mt-6">
                {statCards.map((card, index) => (
                    <div key={index} className="bg-white p-6 rounded-xl shadow-md border border-slate-200 transition-transform transform hover:-translate-y-1">
                        <div className="flex justify-between items-start">
                            <div><p className="text-sm font-medium text-slate-500">{card.title}</p><p className="text-2xl font-bold text-slate-800 mt-1">{card.value}</p></div>
                            <div className={`p-2 rounded-full bg-${card.color}-100 text-${card.color}-600`}><card.icon className="w-6 h-6" /></div>
                        </div>
                    </div>
                ))}
            </div>
            <div className="grid grid-cols-1 lg:grid-cols-3 gap-6 mt-8">
                <div className="lg:col-span-2 bg-white p-6 rounded-xl shadow-md border border-slate-200">
                    <h3 className="text-lg font-semibold text-slate-700">Ringkasan Keuangan</h3>
                    <div className="mt-4 space-y-3">
                        <div className="flex justify-between items-center"><span className="text-slate-600">Total Pemasukan (Cair):</span> <span className="font-bold text-green-600">{formatRupiah(stats.totalPemasukan)}</span></div>
                        <div className="flex justify-between items-center"><span className="text-slate-600">Total Pengeluaran:</span> <span className="font-bold text-red-600">{formatRupiah(stats.totalPengeluaran)}</span></div>
                        <hr/><div className="flex justify-between items-center"><span className="text-slate-800 font-semibold">Saldo:</span> <span className="font-bold text-xl text-slate-800">{formatRupiah(stats.saldoTotal)}</span></div>
                    </div>
                </div>
                <div className="bg-white p-6 rounded-xl shadow-md border border-slate-200">
                    <h3 className="text-lg font-semibold text-slate-700">Biaya Operasional</h3>
                    <div className="mt-4 space-y-3">
                        <div className="flex justify-between items-center"><span className="text-slate-600">Biaya Tenaga Kerja:</span> <span className="font-bold text-slate-800">{formatRupiah(stats.totalUpah)}</span></div>
                        <div className="flex justify-between items-center"><span className="text-slate-600">Biaya Material:</span> <span className="font-bold text-slate-800">{formatRupiah(stats.totalMaterial)}</span></div>
                         <hr/><div className="flex justify-between items-center"><span className="text-slate-800 font-semibold">Total Biaya Operasional:</span> <span className="font-bold text-slate-800">{formatRupiah(totalOperasional)}</span></div>
                    </div>
                </div>
            </div>
        </div>
    );
};

const AnggotaPage = ({ members, openModal, confirmDelete, handleAddOrUpdate }) => {
    const [searchTerm, setSearchTerm] = useState('');
    const [selectedIds, setSelectedIds] = useState([]);
    const filteredMembers = members.filter(member =>
        member.namaLengkap && member.namaLengkap.toLowerCase().includes(searchTerm.toLowerCase())
    );

    const handleSelect = (id) => {
        setSelectedIds(prev => prev.includes(id) ? prev.filter(i => i !== id) : [...prev, id]);
    };

    const handleSelectAll = (e) => {
        if (e.target.checked) {
            setSelectedIds(filteredMembers.map(m => m.id));
        } else {
            setSelectedIds([]);
        }
    };

    return (
        <div>
            <div className="flex justify-between items-center mb-6">
                <div><h2 className="text-3xl font-bold text-slate-800">Anggota POKMAS</h2><p className="text-slate-500 mt-1">Kelola data anggota POKMAS</p></div>
            </div>
            <div className="bg-white p-6 rounded-xl shadow-md border border-slate-200">
                <div className="flex justify-between items-center mb-4">
                    <div className="relative"><Search className="absolute left-3 top-1/2 -translate-y-1/2 text-slate-400 w-5 h-5"/><input type="text" placeholder="Cari berdasarkan nama..." className="pl-10 pr-4 py-2 border border-slate-300 rounded-lg w-64" value={searchTerm} onChange={e => setSearchTerm(e.target.value)} /></div>
                    <div className="flex space-x-2">
                        <button onClick={() => openModal(TambahAnggotaModal, { handleAddOrUpdate })} className="bg-indigo-600 text-white px-4 py-2 rounded-lg shadow hover:bg-indigo-700 transition-all text-sm font-semibold flex items-center"><Plus className="w-4 h-4 mr-2" /> Tambah Anggota</button>
                        <button onClick={() => openModal(TambahAnggotaModal, { dataToEdit: members.find(m => m.id === selectedIds[0]), handleAddOrUpdate })} disabled={selectedIds.length !== 1} className="bg-amber-500 text-white px-4 py-2 rounded-lg shadow hover:bg-amber-600 transition-all text-sm font-semibold flex items-center disabled:opacity-50 disabled:cursor-not-allowed"><Edit className="w-4 h-4 mr-2" /> Edit</button>
                        <button onClick={() => confirmDelete(selectedIds, 'anggota')} disabled={selectedIds.length === 0} className="bg-red-600 text-white px-4 py-2 rounded-lg shadow hover:bg-red-700 transition-all text-sm font-semibold flex items-center disabled:opacity-50 disabled:cursor-not-allowed"><Trash2 className="w-4 h-4 mr-2" /> Hapus</button>
                    </div>
                </div>
                <div className="overflow-x-auto">
                    <table className="w-full text-sm text-left text-slate-500">
                        <thead className="text-xs text-slate-700 uppercase bg-slate-50"><tr><th scope="col" className="p-4"><input type="checkbox" onChange={handleSelectAll} /></th><th scope="col" className="px-6 py-3">Nama</th><th scope="col" className="px-6 py-3">Jabatan</th><th scope="col" className="px-6 py-3">Nomor Telepon</th></tr></thead>
                        <tbody>
                            {filteredMembers.map((member) => (
                                <tr key={member.id} className="bg-white border-b border-slate-200 hover:bg-slate-50">
                                    <td className="p-4"><input type="checkbox" checked={selectedIds.includes(member.id)} onChange={() => handleSelect(member.id)} /></td>
                                    <th scope="row" className="px-6 py-4 font-medium text-slate-900 whitespace-nowrap">{member.namaLengkap}</th>
                                    <td className="px-6 py-4">{member.jabatan}</td><td className="px-6 py-4">{member.nomorTelepon}</td>
                                </tr>
                            ))}
                        </tbody>
                    </table>
                </div>
            </div>
        </div>
    );
};

const KeuanganPage = ({ transactions, workPackages, openModal, confirmDelete, handleAddOrUpdate }) => {
    const [searchTerm, setSearchTerm] = useState('');
    const [selectedIds, setSelectedIds] = useState([]);
    
    const workPackageLocations = new Map(workPackages.map(wp => [wp.namaPaket, wp.lokasi]));

    const filteredTransactions = transactions.filter(t =>
        (t.kategori && t.kategori.toLowerCase().includes(searchTerm.toLowerCase())) ||
        (t.keterangan && t.keterangan.toLowerCase().includes(searchTerm.toLowerCase()))
    );
    const formatRupiah = (number) => new Intl.NumberFormat('id-ID', { style: 'currency', currency: 'IDR', minimumFractionDigits: 0 }).format(number);

    const getStatusClass = (status) => {
        switch (status) {
            case 'Cair': return 'bg-green-100 text-green-800';
            case 'Direncanakan': return 'bg-yellow-100 text-yellow-800';
            default: return 'bg-slate-100 text-slate-800';
        }
    };
    
    const totalPagu = workPackages.reduce((sum, wp) => sum + Number(wp.nilaiPagu || 0), 0);
    const totalTahap1 = totalPagu * 0.4;
    const totalTahap2 = totalPagu * 0.3;
    const totalTahap3 = totalPagu * 0.3;
    const totalDanaCair = transactions.filter(t => t.jenis === 'Pemasukan' && t.status === 'Cair').reduce((sum, t) => sum + t.jumlah, 0);

    const handleSelect = (id) => {
        setSelectedIds(prev => prev.includes(id) ? prev.filter(i => i !== id) : [...prev, id]);
    };

    const handleSelectAll = (e) => {
        if (e.target.checked) {
            setSelectedIds(filteredTransactions.map(t => t.id));
        } else {
            setSelectedIds([]);
        }
    };

    return (
        <div>
            <div className="flex justify-between items-center mb-6">
                <div><h2 className="text-3xl font-bold text-slate-800">Keuangan</h2><p className="text-slate-500 mt-1">Kelola transaksi pemasukan dan pengeluaran</p></div>
            </div>
            
            <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-6 mb-8">
                <div className="bg-white p-4 rounded-xl shadow-md border border-slate-200">
                    <p className="text-sm font-medium text-slate-500">Total Dana Tahap 1</p>
                    <p className="text-xl font-bold text-slate-800 mt-1">{formatRupiah(totalTahap1)}</p>
                </div>
                <div className="bg-white p-4 rounded-xl shadow-md border border-slate-200">
                    <p className="text-sm font-medium text-slate-500">Total Dana Tahap 2</p>
                    <p className="text-xl font-bold text-slate-800 mt-1">{formatRupiah(totalTahap2)}</p>
                </div>
                <div className="bg-white p-4 rounded-xl shadow-md border border-slate-200">
                    <p className="text-sm font-medium text-slate-500">Total Dana Tahap 3</p>
                    <p className="text-xl font-bold text-slate-800 mt-1">{formatRupiah(totalTahap3)}</p>
                </div>
                <div className="bg-green-50 p-4 rounded-xl shadow-md border border-green-200">
                    <p className="text-sm font-medium text-green-700">Total Semua Dana Cair</p>
                    <p className="text-xl font-bold text-green-800 mt-1">{formatRupiah(totalDanaCair)}</p>
                </div>
            </div>

            <div className="mt-6 bg-white p-6 rounded-xl shadow-md border border-slate-200">
                <div className="flex justify-between items-center mb-4">
                    <div className="relative"><Search className="absolute left-3 top-1/2 -translate-y-1/2 text-slate-400 w-5 h-5"/><input type="text" placeholder="Cari kategori/keterangan..." className="pl-10 pr-4 py-2 border border-slate-300 rounded-lg w-64" value={searchTerm} onChange={e => setSearchTerm(e.target.value)} /></div>
                    <div className="flex space-x-2">
                        <button onClick={() => openModal(TambahTransaksiModal, { workPackages, handleAddOrUpdate, transactions, dataToEdit: null })} className="bg-indigo-600 text-white px-4 py-2 rounded-lg shadow hover:bg-indigo-700 transition-all text-sm font-semibold flex items-center"><Plus className="w-4 h-4 mr-2" /> Tambah Transaksi</button>
                        <button onClick={() => openModal(TambahTransaksiModal, { dataToEdit: transactions.find(t => t.id === selectedIds[0]), workPackages, handleAddOrUpdate, transactions })} disabled={selectedIds.length !== 1} className="bg-amber-500 text-white px-4 py-2 rounded-lg shadow hover:bg-amber-600 transition-all text-sm font-semibold flex items-center disabled:opacity-50 disabled:cursor-not-allowed"><Edit className="w-4 h-4 mr-2" /> Edit</button>
                        <button onClick={() => confirmDelete(selectedIds, 'transaksi')} disabled={selectedIds.length === 0} className="bg-red-600 text-white px-4 py-2 rounded-lg shadow hover:bg-red-700 transition-all text-sm font-semibold flex items-center disabled:opacity-50 disabled:cursor-not-allowed"><Trash2 className="w-4 h-4 mr-2" /> Hapus</button>
                    </div>
                </div>
                <div className="overflow-x-auto mt-4">
                    <table className="w-full text-sm text-left text-slate-500">
                        <thead className="text-xs text-slate-700 uppercase bg-slate-50">
                            <tr>
                                <th scope="col" className="p-4"><input type="checkbox" onChange={handleSelectAll} /></th>
                                <th scope="col" className="px-6 py-3">Tanggal</th>
                                <th scope="col" className="px-6 py-3">Jenis</th>
                                <th scope="col" className="px-6 py-3">Kategori</th>
                                <th scope="col" className="px-6 py-3">Paket Pekerjaan</th>
                                <th scope="col" className="px-6 py-3">Lokasi</th>
                                <th scope="col" className="px-6 py-3">Jumlah</th>
                                <th scope="col" className="px-6 py-3">Status</th>
                            </tr>
                        </thead>
                        <tbody>
                            {filteredTransactions.map((t) => {
                                const lokasi = workPackageLocations.get(t.paketPekerjaan) || '-';
                                return (
                                <tr key={t.id} className="bg-white border-b border-slate-200 hover:bg-slate-50">
                                    <td className="p-4"><input type="checkbox" checked={selectedIds.includes(t.id)} onChange={() => handleSelect(t.id)} /></td>
                                    <td className="px-6 py-4">{t.tanggal}</td>
                                    <td className="px-6 py-4"><span className={`px-2 py-1 rounded-full text-xs font-semibold ${t.jenis === 'Pemasukan' ? 'bg-green-100 text-green-800' : 'bg-red-100 text-red-800'}`}>{t.jenis}</span></td>
                                    <td className="px-6 py-4">{t.kategori}</td>
                                    <td className="px-6 py-4">{t.paketPekerjaan || '-'}</td>
                                    <td className="px-6 py-4">{lokasi}</td>
                                    <td className="px-6 py-4 font-medium text-slate-900">{formatRupiah(t.jumlah)}</td>
                                    <td className="px-6 py-4"><span className={`px-2 py-1 rounded-full text-xs font-semibold ${getStatusClass(t.status)}`}>{t.status || 'Cair'}</span></td>
                                </tr>
                                );
                            })}
                        </tbody>
                    </table>
                </div>
            </div>
        </div>
    );
};

const RincianTenagaKerja = ({ wageBills, workPackages }) => {
    const formatRupiah = (number) => new Intl.NumberFormat('id-ID', { style: 'currency', currency: 'IDR', minimumFractionDigits: 0 }).format(number);

    const workersData = wageBills.reduce((acc, bill) => {
        if (!bill.namaPekerja) return acc; // Safety check
        const workerName = bill.namaPekerja;
        if (!acc[workerName]) {
            acc[workerName] = {
                bills: [],
                totalAmount: 0,
            };
        }
        
        const wp = workPackages.find(p => p.namaPaket === bill.paketPekerjaan);
        const location = wp ? wp.lokasi : 'N/A';

        acc[workerName].bills.push({ ...bill, lokasi: location });
        acc[workerName].totalAmount += Number(bill.jumlah);
        
        return acc;
    }, {});

    const sortedWorkers = Object.entries(workersData)
        .map(([nama, data]) => ({ nama, ...data }))
        .sort((a, b) => b.totalAmount - a.totalAmount);

    return (
        <div className="mt-6 space-y-4">
            {sortedWorkers.length === 0 ? (
                <div className="text-center py-10 text-slate-500 bg-white rounded-xl shadow-md border border-slate-200">
                    <ClipboardList className="mx-auto w-12 h-12 text-slate-400" />
                    <h3 className="mt-2 text-lg font-semibold">Belum Ada Data Tagihan Upah</h3>
                    <p className="mt-1 text-sm">Data akan muncul di sini setelah Anda menambahkan tagihan upah di menu 'Tagihan Upah'.</p>
                </div>
            ) : sortedWorkers.map(worker => (
                <div key={worker.nama} className="bg-white p-4 rounded-lg shadow-sm border border-slate-200">
                    <div className="flex justify-between items-center">
                        <h4 className="font-bold text-slate-800">{worker.nama}</h4>
                        <span className="text-lg font-bold text-indigo-600">{formatRupiah(worker.totalAmount)}</span>
                    </div>
                    <div className="mt-4 overflow-x-auto">
                        <table className="w-full text-sm text-left text-slate-500">
                            <thead className="text-xs text-slate-700 uppercase bg-slate-50">
                                <tr>
                                    <th scope="col" className="px-4 py-2">Paket Pekerjaan</th>
                                    <th scope="col" className="px-4 py-2">Lokasi</th>
                                    <th scope="col" className="px-4 py-2">Tanggal</th>
                                    <th scope="col" className="px-4 py-2">Jumlah</th>
                                    <th scope="col" className="px-4 py-2">Status</th>
                                </tr>
                            </thead>
                            <tbody>
                                {worker.bills.sort((a, b) => new Date(b.tanggal) - new Date(a.tanggal)).map((bill) => (
                                    <tr key={bill.id} className="border-b last:border-b-0 hover:bg-slate-50">
                                        <td className="px-4 py-2 font-medium text-slate-800">{Array.isArray(bill.paketPekerjaan) ? bill.paketPekerjaan.join(', ') : bill.paketPekerjaan}</td>
                                        <td className="px-4 py-2">{bill.lokasi}</td>
                                        <td className="px-4 py-2">{bill.tanggal}</td>
                                        <td className="px-4 py-2">{formatRupiah(bill.jumlah)}</td>
                                        <td className="px-4 py-2">
                                            <span className={`px-2 py-1 rounded-full text-xs font-semibold ${bill.status === 'Lunas' ? 'bg-green-100 text-green-800' : 'bg-yellow-100 text-yellow-800'}`}>{bill.status}</span>
                                        </td>
                                    </tr>
                                ))}
                            </tbody>
                        </table>
                    </div>
                </div>
            ))}
        </div>
    );
};


const PaketPekerjaanPage = ({ workPackages, wageBills, transactions, openModal, confirmDelete, handleAddOrUpdate, handleFileDelete, handleFileUpload, handleBulkAdd, confirmReset, handleDisburseFromPackagePage }) => {
    const [activeTab, setActiveTab] = useState('Daftar Paket');
    const [searchTerm, setSearchTerm] = useState('');
    const [selectedIds, setSelectedIds] = useState([]);
    const fileInputRefs = useRef({});

    const filteredWorkPackages = workPackages.filter(wp =>
        (wp.namaPaket && wp.namaPaket.toLowerCase().includes(searchTerm.toLowerCase())) ||
        (wp.lokasi && wp.lokasi.toLowerCase().includes(searchTerm.toLowerCase()))
    );
    const formatRupiah = (number) => new Intl.NumberFormat('id-ID', { style: 'currency', currency: 'IDR', minimumFractionDigits: 0 }).format(number);
    const getStatusClass = (status) => {
        switch (status) {
            case 'Berjalan': return 'bg-blue-100 text-blue-800';
            case 'Selesai': return 'bg-green-100 text-green-800';
            case 'Direncanakan': return 'bg-yellow-100 text-yellow-800';
            default: return 'bg-slate-100 text-slate-800';
        }
    };
    
    const handleSelect = (id) => {
        setSelectedIds(prev => prev.includes(id) ? prev.filter(i => i !== id) : [...prev, id]);
    };

    const handleSelectAll = (e) => {
        if (e.target.checked) {
            setSelectedIds(filteredWorkPackages.map(wp => wp.id));
        } else {
            setSelectedIds([]);
        }
    };
    
    const confirmFileDelete = (packageId, fileName) => {
        openModal(ConfirmationModal, {
            onConfirm: () => handleFileDelete(packageId, fileName),
            title: `Hapus File Lampiran`,
            message: `Apakah Anda yakin ingin menghapus file "${fileName}"? Tindakan ini tidak dapat dibatalkan.`
        });
    };

    const triggerFileInput = (packageId) => {
        fileInputRefs.current[packageId].click();
    };

    const onFileSelected = (e, packageId) => {
        const file = e.target.files[0];
        if (file) {
            handleFileUpload(packageId, file);
        }
    };

    return (
        <div>
            <div className="flex justify-between items-center mb-6">
                <div><h2 className="text-3xl font-bold text-slate-800">Paket Pekerjaan</h2><p className="text-slate-500 mt-1">Kelola dan pantau kemajuan proyek</p></div>
            </div>
            
            <div className="border-b border-slate-200">
                <nav className="-mb-px flex space-x-6" aria-label="Tabs">
                    <button
                        onClick={() => setActiveTab('Daftar Paket')}
                        className={`whitespace-nowrap py-3 px-1 border-b-2 font-medium text-sm ${activeTab === 'Daftar Paket' ? 'border-indigo-500 text-indigo-600' : 'border-transparent text-slate-500 hover:text-slate-700 hover:border-slate-300'}`}
                    >
                        Daftar Paket
                    </button>
                    <button
                        onClick={() => setActiveTab('Rincian Tenaga Kerja')}
                        className={`whitespace-nowrap py-3 px-1 border-b-2 font-medium text-sm ${activeTab === 'Rincian Tenaga Kerja' ? 'border-indigo-500 text-indigo-600' : 'border-transparent text-slate-500 hover:text-slate-700 hover:border-slate-300'}`}
                    >
                        Rincian Tenaga Kerja
                    </button>
                </nav>
            </div>

            {activeTab === 'Daftar Paket' && (
                <div className="mt-6 bg-white p-6 rounded-xl shadow-md border border-slate-200">
                    <div className="flex justify-between items-center mb-4">
                        <div className="relative"><Search className="absolute left-3 top-1/2 -translate-y-1/2 text-slate-400 w-5 h-5"/><input type="text" placeholder="Cari nama/lokasi paket..." className="pl-10 pr-4 py-2 border border-slate-300 rounded-lg w-64" value={searchTerm} onChange={e => setSearchTerm(e.target.value)} /></div>
                         <div className="flex space-x-2">
                            <button onClick={() => openModal(UploadModal, { type: 'paket', handleBulkAdd })} className="bg-teal-600 text-white px-4 py-2 rounded-lg shadow hover:bg-teal-700 transition-all text-sm font-semibold flex items-center"><Upload className="w-4 h-4 mr-2" /> Upload Data</button>
                            <button onClick={() => openModal(TambahPaketPekerjaanModal, { handleAddOrUpdate })} className="bg-indigo-600 text-white px-4 py-2 rounded-lg shadow hover:bg-indigo-700 transition-all text-sm font-semibold flex items-center"><Plus className="w-4 h-4 mr-2" /> Tambah Paket</button>
                            <button onClick={() => openModal(TambahPaketPekerjaanModal, { dataToEdit: workPackages.find(wp => wp.id === selectedIds[0]), handleAddOrUpdate })} disabled={selectedIds.length !== 1} className="bg-amber-500 text-white px-4 py-2 rounded-lg shadow hover:bg-amber-600 transition-all text-sm font-semibold flex items-center disabled:opacity-50 disabled:cursor-not-allowed"><Edit className="w-4 h-4 mr-2" /> Edit</button>
                            <button onClick={() => confirmDelete(selectedIds, 'paket')} disabled={selectedIds.length === 0} className="bg-red-600 text-white px-4 py-2 rounded-lg shadow hover:bg-red-700 transition-all text-sm font-semibold flex items-center disabled:opacity-50 disabled:cursor-not-allowed"><Trash2 className="w-4 h-4 mr-2" /> Hapus</button>
                            <button onClick={confirmReset} className="bg-slate-600 text-white px-4 py-2 rounded-lg shadow hover:bg-slate-700 transition-all text-sm font-semibold flex items-center"><RefreshCw className="w-4 h-4 mr-2" /> Reset</button>
                        </div>
                    </div>
                    <div className="overflow-x-auto mt-4">
                        <table className="w-full text-sm text-left text-slate-500">
                            <thead className="text-xs text-slate-700 uppercase bg-slate-50">
                                <tr>
                                    <th scope="col" className="p-4"><input type="checkbox" onChange={handleSelectAll} /></th>
                                    <th scope="col" className="px-6 py-3">Nama Paket</th>
                                    <th scope="col" className="px-6 py-3">Dana Cair & Progress</th>
                                    <th scope="col" className="px-6 py-3">File Lampiran</th>
                                    <th scope="col" className="px-6 py-3">Aksi Pencairan</th>
                                    <th scope="col" className="px-6 py-3">Status</th>
                                </tr>
                            </thead>
                            <tbody>
                                {filteredWorkPackages.map((wp) => {
                                    fileInputRefs.current[wp.id] = fileInputRefs.current[wp.id] || React.createRef();
                                    const danaCair = transactions
                                        .filter(t => t.paketPekerjaan === wp.namaPaket && t.status === 'Cair' && t.jenis === 'Pemasukan')
                                        .reduce((sum, t) => sum + t.jumlah, 0);
                                    const progress = wp.nilaiPagu > 0 ? (danaCair / wp.nilaiPagu) * 100 : 0;
                                    
                                    const isTahap1Cair = transactions.some(t => t.paketPekerjaan === wp.namaPaket && t.kategori.includes('Tahap 1') && t.status === 'Cair');
                                    const isTahap2Cair = transactions.some(t => t.paketPekerjaan === wp.namaPaket && t.kategori.includes('Tahap 2') && t.status === 'Cair');
                                    const isTahap3Cair = transactions.some(t => t.paketPekerjaan === wp.namaPaket && t.kategori.includes('Tahap 3') && t.status === 'Cair');

                                    return (
                                        <tr key={wp.id} className="bg-white border-b border-slate-200 hover:bg-slate-50">
                                            <td className="p-4"><input type="checkbox" checked={selectedIds.includes(wp.id)} onChange={() => handleSelect(wp.id)} /></td>
                                            <th scope="row" className="px-6 py-4 font-medium text-slate-900 whitespace-nowrap">
                                                <div>{wp.namaPaket}</div>
                                                <div className="text-xs text-slate-500 font-normal">{wp.lokasi}</div>
                                                <div className="text-xs text-slate-500 font-normal">{formatRupiah(wp.nilaiPagu)}</div>
                                            </th>
                                            <td className="px-6 py-4">
                                                <div className="font-medium">{formatRupiah(danaCair)}</div>
                                                <div className="w-full bg-slate-200 rounded-full h-2.5 mt-1">
                                                    <div className="bg-indigo-600 h-2.5 rounded-full" style={{ width: `${progress.toFixed(2)}%` }}></div>
                                                </div>
                                            </td>
                                            <td className="px-6 py-4">
                                                <input type="file" ref={el => fileInputRefs.current[wp.id] = el} onChange={(e) => onFileSelected(e, wp.id)} className="hidden" />
                                                {wp.fileURL ? (
                                                    <div className="flex items-center space-x-2">
                                                        <a href={wp.fileURL} target="_blank" rel="noopener noreferrer" className="text-indigo-600 hover:underline flex items-center text-xs font-semibold">
                                                            <DownloadCloud className="w-4 h-4 mr-1" /> {wp.fileName}
                                                        </a>
                                                        <button onClick={() => triggerFileInput(wp.id)} className="text-amber-500 hover:text-amber-700"><Replace className="w-4 h-4" /></button>
                                                        <button onClick={() => confirmFileDelete(wp.id, wp.fileName)} className="text-red-500 hover:text-red-700"><XCircle className="w-4 h-4" /></button>
                                                    </div>
                                                ) : (
                                                    <button onClick={() => triggerFileInput(wp.id)} className="text-xs bg-slate-100 hover:bg-slate-200 text-slate-600 font-semibold py-1 px-3 rounded-full flex items-center">
                                                        <Upload className="w-3 h-3 mr-1" /> Upload
                                                    </button>
                                                )}
                                            </td>
                                            <td className="px-6 py-4">
                                                <div className="flex flex-col space-y-1">
                                                     <button onClick={() => handleDisburseFromPackagePage(wp, 'Tahap 1')} disabled={isTahap1Cair} className="text-xs bg-blue-100 text-blue-700 font-semibold py-1 px-2 rounded-full flex items-center justify-center disabled:bg-slate-100 disabled:text-slate-400 disabled:cursor-not-allowed">
                                                        {isTahap1Cair && <CheckCircle className="w-3 h-3 mr-1" />} Tahap 1
                                                     </button>
                                                     <button onClick={() => handleDisburseFromPackagePage(wp, 'Tahap 2')} disabled={isTahap2Cair} className="text-xs bg-blue-100 text-blue-700 font-semibold py-1 px-2 rounded-full flex items-center justify-center disabled:bg-slate-100 disabled:text-slate-400 disabled:cursor-not-allowed">
                                                        {isTahap2Cair && <CheckCircle className="w-3 h-3 mr-1" />} Tahap 2
                                                     </button>
                                                     <button onClick={() => handleDisburseFromPackagePage(wp, 'Tahap 3')} disabled={isTahap3Cair} className="text-xs bg-blue-100 text-blue-700 font-semibold py-1 px-2 rounded-full flex items-center justify-center disabled:bg-slate-100 disabled:text-slate-400 disabled:cursor-not-allowed">
                                                        {isTahap3Cair && <CheckCircle className="w-3 h-3 mr-1" />} Tahap 3
                                                     </button>
                                                </div>
                                            </td>
                                            <td className="px-6 py-4"><span className={`px-2 py-1 rounded-full text-xs font-semibold ${getStatusClass(wp.status)}`}>{wp.status}</span></td>
                                        </tr>
                                    );
                                })}
                            </tbody>
                        </table>
                    </div>
                </div>
            )}
            
            {activeTab === 'Rincian Tenaga Kerja' && (
                <RincianTenagaKerja wageBills={wageBills} workPackages={workPackages} />
            )}
        </div>
    );
};

const TagihanUpahPage = ({ wageBills, openModal, workPackages, confirmDelete, handleAddOrUpdate }) => {
    const [activeTab, setActiveTab] = useState('Daftar Tagihan');
    const [searchTerm, setSearchTerm] = useState('');
    const [selectedIds, setSelectedIds] = useState([]);
    const [filterStatus, setFilterStatus] = useState('Semua');
    const [sortConfig, setSortConfig] = useState({ key: 'tanggal', direction: 'descending' });

    const sortedAndFilteredBills = React.useMemo(() => {
        let bills = [...wageBills];

        if (filterStatus !== 'Semua') {
            bills = bills.filter(bill => bill.status === filterStatus);
        }

        if (searchTerm) {
            bills = bills.filter(bill =>
                (bill.namaPekerja && bill.namaPekerja.toLowerCase().includes(searchTerm.toLowerCase())) ||
                (Array.isArray(bill.paketPekerjaan) ? bill.paketPekerjaan.join(' ').toLowerCase().includes(searchTerm.toLowerCase()) : (bill.paketPekerjaan || '').toLowerCase().includes(searchTerm.toLowerCase()))
            );
        }

        if (sortConfig.key) {
            bills.sort((a, b) => {
                let aValue = a[sortConfig.key];
                let bValue = b[sortConfig.key];

                if (aValue === undefined || aValue === null) return 1;
                if (bValue === undefined || bValue === null) return -1;

                if (sortConfig.key === 'jumlah') {
                    aValue = Number(aValue);
                    bValue = Number(bValue);
                }
                
                if (aValue < bValue) {
                    return sortConfig.direction === 'ascending' ? -1 : 1;
                }
                if (aValue > bValue) {
                    return sortConfig.direction === 'ascending' ? 1 : -1;
                }
                return 0;
            });
        }

        return bills;
    }, [wageBills, searchTerm, filterStatus, sortConfig]);

    const requestSort = (key) => {
        let direction = 'ascending';
        if (sortConfig.key === key && sortConfig.direction === 'ascending') {
            direction = 'descending';
        }
        setSortConfig({ key, direction });
    };

    const getSortIndicator = (key) => {
        if (sortConfig.key !== key) return null;
        return sortConfig.direction === 'ascending' ? ' ▲' : ' ▼';
    };

    const formatRupiah = (number) => new Intl.NumberFormat('id-ID', { style: 'currency', currency: 'IDR', minimumFractionDigits: 0 }).format(number);
    
    const handleSelect = (id) => {
        setSelectedIds(prev => prev.includes(id) ? prev.filter(i => i !== id) : [...prev, id]);
    };

    const handleSelectAll = (e) => {
        if (e.target.checked) {
            setSelectedIds(sortedAndFilteredBills.map(b => b.id));
        } else {
            setSelectedIds([]);
        }
    };

    const SortableHeader = ({ label, columnKey }) => (
        <th scope="col" className="px-6 py-3">
            <button onClick={() => requestSort(columnKey)} className="flex items-center space-x-1 font-bold">
                <span>{label}</span>
                <span>{getSortIndicator(columnKey)}</span>
            </button>
        </th>
    );

    return (
        <div>
            <div className="flex justify-between items-center mb-6">
                <div><h2 className="text-3xl font-bold text-slate-800">Tagihan Upah Tenaga Kerja</h2><p className="text-slate-500 mt-1">Kelola tagihan upah harian dan borongan</p></div>
            </div>

            <div className="border-b border-slate-200">
                <nav className="-mb-px flex space-x-6" aria-label="Tabs">
                    <button onClick={() => setActiveTab('Daftar Tagihan')} className={`whitespace-nowrap py-3 px-1 border-b-2 font-medium text-sm ${activeTab === 'Daftar Tagihan' ? 'border-indigo-500 text-indigo-600' : 'border-transparent text-slate-500 hover:text-slate-700 hover:border-slate-300'}`}>
                        Daftar Tagihan
                    </button>
                    <button onClick={() => setActiveTab('Tambah Rincian Tagihan')} className={`whitespace-nowrap py-3 px-1 border-b-2 font-medium text-sm ${activeTab === 'Tambah Rincian Tagihan' ? 'border-indigo-500 text-indigo-600' : 'border-transparent text-slate-500 hover:text-slate-700 hover:border-slate-300'}`}>
                        Tambah Rincian Tagihan
                    </button>
                </nav>
            </div>

            {activeTab === 'Daftar Tagihan' && (
                <div className="mt-6 bg-white p-6 rounded-xl shadow-md border border-slate-200">
                     <div className="flex justify-between items-center mb-4">
                        <div className="flex items-center space-x-4">
                            <div className="relative"><Search className="absolute left-3 top-1/2 -translate-y-1/2 text-slate-400 w-5 h-5"/><input type="text" placeholder="Cari nama pekerja..." className="pl-10 pr-4 py-2 border border-slate-300 rounded-lg w-64" value={searchTerm} onChange={e => setSearchTerm(e.target.value)} /></div>
                            <FormSelect id="statusFilter" value={filterStatus} onChange={(e) => setFilterStatus(e.target.value)}>
                                <option value="Semua">Semua Status</option>
                                <option value="Lunas">Lunas</option>
                                <option value="Belum Dibayar">Belum Dibayar</option>
                            </FormSelect>
                        </div>
                        <div className="flex space-x-2">
                            <button onClick={() => openModal(TambahTagihanUpahModal, { workPackages, handleAddOrUpdate })} className="bg-indigo-600 text-white px-4 py-2 rounded-lg shadow hover:bg-indigo-700 transition-all text-sm font-semibold flex items-center"><Plus className="w-4 h-4 mr-2" /> Tambah Tagihan Umum</button>
                            <button 
                                onClick={() => {
                                    const selectedBill = wageBills.find(b => b.id === selectedIds[0]);
                                    if (selectedBill) {
                                        openModal(TambahTagihanUpahModal, { dataToEdit: selectedBill, workPackages, handleAddOrUpdate });
                                    }
                                }} 
                                disabled={selectedIds.length !== 1 || Array.isArray(wageBills.find(b => b.id === selectedIds[0])?.paketPekerjaan)} 
                                className="bg-amber-500 text-white px-4 py-2 rounded-lg shadow hover:bg-amber-600 transition-all text-sm font-semibold flex items-center disabled:opacity-50 disabled:cursor-not-allowed"
                            >
                                <Edit className="w-4 h-4 mr-2" /> Edit
                            </button>
                            <button onClick={() => confirmDelete(selectedIds, 'tagihan-upah')} disabled={selectedIds.length === 0} className="bg-red-600 text-white px-4 py-2 rounded-lg shadow hover:bg-red-700 transition-all text-sm font-semibold flex items-center disabled:opacity-50 disabled:cursor-not-allowed"><Trash2 className="w-4 h-4 mr-2" /> Hapus</button>
                        </div>
                    </div>
                    <div className="overflow-x-auto mt-4">
                        <table className="w-full text-sm text-left text-slate-500">
                            <thead className="text-xs text-slate-700 uppercase bg-slate-50">
                                <tr>
                                    <th scope="col" className="p-4"><input type="checkbox" onChange={handleSelectAll} /></th>
                                    <SortableHeader label="Nama Pekerja" columnKey="namaPekerja" />
                                    <SortableHeader label="Tanggal" columnKey="tanggal" />
                                    <SortableHeader label="Kategori" columnKey="kategori" />
                                    <SortableHeader label="Paket Pekerjaan" columnKey="paketPekerjaan" />
                                    <SortableHeader label="Total" columnKey="jumlah" />
                                    <SortableHeader label="Status" columnKey="status" />
                                </tr>
                            </thead>
                            <tbody>
                                {sortedAndFilteredBills.map((bill) => (
                                    <tr key={bill.id} className="bg-white border-b border-slate-200 hover:bg-slate-50">
                                        <td className="p-4"><input type="checkbox" checked={selectedIds.includes(bill.id)} onChange={() => handleSelect(bill.id)} /></td>
                                        <th scope="row" className="px-6 py-4 font-medium text-slate-900 whitespace-nowrap">{bill.namaPekerja}</th>
                                        <td className="px-6 py-4">{bill.tanggal}</td>
                                        <td className="px-6 py-4">{bill.kategori}</td>
                                        <td className="px-6 py-4">{Array.isArray(bill.paketPekerjaan) ? bill.paketPekerjaan.join(', ') : bill.paketPekerjaan}</td>
                                        <td className="px-6 py-4 font-medium">{formatRupiah(bill.jumlah)}</td>
                                        <td className="px-6 py-4">
                                            <span className={`px-2 py-1 rounded-full text-xs font-semibold ${bill.status === 'Lunas' ? 'bg-green-100 text-green-800' : 'bg-yellow-100 text-yellow-800'}`}>{bill.status}</span>
                                        </td>
                                    </tr>
                                ))}
                            </tbody>
                        </table>
                    </div>
                </div>
            )}
            {activeTab === 'Tambah Rincian Tagihan' && (
                <TambahRincianTagihanForm workPackages={workPackages} handleAddOrUpdate={handleAddOrUpdate} setActiveTab={setActiveTab} />
            )}
        </div>
    );
};

const TagihanMaterialPage = ({ materialBills, openModal, workPackages, confirmDelete, handleAddOrUpdate }) => {
    const [searchTerm, setSearchTerm] = useState('');
    const [selectedIds, setSelectedIds] = useState([]);
    const filteredMaterialBills = materialBills.filter(bill =>
        (bill.toko && bill.toko.toLowerCase().includes(searchTerm.toLowerCase())) ||
        (bill.uraian && bill.uraian.toLowerCase().includes(searchTerm.toLowerCase()))
    );
    const formatRupiah = (number) => new Intl.NumberFormat('id-ID', { style: 'currency', currency: 'IDR', minimumFractionDigits: 0 }).format(number);
    
    const handleSelect = (id) => {
        setSelectedIds(prev => prev.includes(id) ? prev.filter(i => i !== id) : [...prev, id]);
    };

    const handleSelectAll = (e) => {
        if (e.target.checked) {
            setSelectedIds(filteredMaterialBills.map(b => b.id));
        } else {
            setSelectedIds([]);
        }
    };

    return (
        <div>
            <div className="flex justify-between items-center mb-6">
                <div><h2 className="text-3xl font-bold text-slate-800">Tagihan Material</h2><p className="text-slate-500 mt-1">Kelola tagihan pembelian material</p></div>
            </div>
            <div className="mt-6 bg-white p-6 rounded-xl shadow-md border border-slate-200">
                <div className="flex justify-between items-center mb-4">
                    <div className="relative"><Search className="absolute left-3 top-1/2 -translate-y-1/2 text-slate-400 w-5 h-5"/><input type="text" placeholder="Cari toko/uraian..." className="pl-10 pr-4 py-2 border border-slate-300 rounded-lg w-64" value={searchTerm} onChange={e => setSearchTerm(e.target.value)} /></div>
                    <div className="flex space-x-2">
                        <button onClick={() => openModal(TambahTagihanMaterialModal, { workPackages, handleAddOrUpdate })} className="bg-indigo-600 text-white px-4 py-2 rounded-lg shadow hover:bg-indigo-700 transition-all text-sm font-semibold flex items-center"><Plus className="w-4 h-4 mr-2" /> Tambah Tagihan</button>
                        <button onClick={() => openModal(TambahTagihanMaterialModal, { dataToEdit: materialBills.find(b => b.id === selectedIds[0]), workPackages, handleAddOrUpdate })} disabled={selectedIds.length !== 1} className="bg-amber-500 text-white px-4 py-2 rounded-lg shadow hover:bg-amber-600 transition-all text-sm font-semibold flex items-center disabled:opacity-50 disabled:cursor-not-allowed"><Edit className="w-4 h-4 mr-2" /> Edit</button>
                        <button onClick={() => confirmDelete(selectedIds, 'tagihan-material')} disabled={selectedIds.length === 0} className="bg-red-600 text-white px-4 py-2 rounded-lg shadow hover:bg-red-700 transition-all text-sm font-semibold flex items-center disabled:opacity-50 disabled:cursor-not-allowed"><Trash2 className="w-4 h-4 mr-2" /> Hapus</button>
                    </div>
                </div>
                <div className="overflow-x-auto mt-4">
                    <table className="w-full text-sm text-left text-slate-500">
                        <thead className="text-xs text-slate-700 uppercase bg-slate-50">
                            <tr>
                                <th scope="col" className="p-4"><input type="checkbox" onChange={handleSelectAll} /></th>
                                <th scope="col" className="px-6 py-3">Toko</th>
                                <th scope="col" className="px-6 py-3">Uraian</th>
                                <th scope="col" className="px-6 py-3">Total</th>
                                <th scope="col" className="px-6 py-3">Tanggal</th>
                                <th scope="col" className="px-6 py-3">Status</th>
                            </tr>
                        </thead>
                        <tbody>
                            {filteredMaterialBills.map((bill) => (
                                <tr key={bill.id} className="bg-white border-b border-slate-200 hover:bg-slate-50">
                                    <td className="p-4"><input type="checkbox" checked={selectedIds.includes(bill.id)} onChange={() => handleSelect(bill.id)} /></td>
                                    <th scope="row" className="px-6 py-4 font-medium text-slate-900 whitespace-nowrap">{bill.toko}</th>
                                    <td className="px-6 py-4">{bill.uraian}</td>
                                    <td className="px-6 py-4 font-medium">{formatRupiah(bill.total)}</td>
                                    <td className="px-6 py-4">{bill.tanggal}</td>
                                    <td className="px-6 py-4">
                                        <span className={`px-2 py-1 rounded-full text-xs font-semibold ${bill.status === 'Lunas' ? 'bg-green-100 text-green-800' : 'bg-yellow-100 text-yellow-800'}`}>{bill.status}</span>
                                    </td>
                                </tr>
                            ))}
                        </tbody>
                    </table>
                </div>
            </div>
        </div>
    );
};

const LaporanPage = ({ data, stats }) => {
    const { members, workPackages, transactions, wageBills, materialBills } = data;
    const [analysis, setAnalysis] = useState('');
    const [isAnalyzing, setIsAnalyzing] = useState(false);
    const formatRupiah = (number) => new Intl.NumberFormat('id-ID', { style: 'currency', currency: 'IDR', minimumFractionDigits: 0 }).format(number);

    const handleGenerateAnalysis = async () => {
        setIsAnalyzing(true);
        setAnalysis('');
        const prompt = `
            Anda adalah seorang analis keuangan untuk sebuah Kelompok Masyarakat (POKMAS) di Indonesia.
            Berikut adalah ringkasan keuangan POKMAS Karya 25:
            - Total Pemasukan (yang sudah cair): ${formatRupiah(stats.totalPemasukan)}
            - Total Pengeluaran: ${formatRupiah(stats.totalPengeluaran)}
            - Saldo Kas Saat Ini: ${formatRupiah(stats.saldoTotal)}
            - Total Biaya Upah Tenaga Kerja: ${formatRupiah(stats.totalUpah)}
            - Total Biaya Material: ${formatRupiah(stats.totalMaterial)}

            Berdasarkan data di atas, berikan analisis singkat dan saran keuangan dalam 3 poin utama dalam format markdown. Fokus pada kesehatan keuangan, efisiensi pengeluaran, dan saran untuk masa depan. Gunakan bahasa yang formal dan mudah dimengerti.
        `;
        try {
            const result = await callGeminiApi(prompt);
            setAnalysis(result);
        } catch (error) {
            setAnalysis("Gagal membuat analisis. Silakan coba lagi.");
        } finally {
            setIsAnalyzing(false);
        }
    };

    const exportToCSV = (dataToExport, filename) => {
        if (!dataToExport || dataToExport.length === 0) {
            alert("Tidak ada data untuk diekspor.");
            return;
        }
        
        const headers = Object.keys(dataToExport[0]);
        const csvRows = [
            headers.join(','),
            ...dataToExport.map(row =>
                headers.map(fieldName => JSON.stringify(row[fieldName])).join(',')
            )
        ];

        const blob = new Blob([csvRows.join('\n')], { type: 'text/csv;charset=utf-8;' });
        const link = document.createElement('a');
        const url = URL.createObjectURL(blob);
        link.setAttribute('href', url);
        link.setAttribute('download', `${filename}.csv`);
        link.style.visibility = 'hidden';
        document.body.appendChild(link);
        link.click();
        document.body.removeChild(link);
    };

    return (
        <div>
            <h2 className="text-3xl font-bold text-slate-800">Laporan & Analisis</h2>
            <p className="text-slate-500 mt-1">Unduh data atau buat analisis keuangan otomatis.</p>

            <div className="mt-8 grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
                <div className="bg-white p-6 rounded-xl shadow-md border border-slate-200 lg:col-span-3">
                    <h3 className="text-lg font-semibold text-slate-700">✨ Analisis Keuangan Otomatis</h3>
                    <p className="text-sm text-slate-500 mt-1">Dapatkan wawasan dan saran berdasarkan data keuangan Anda saat ini.</p>
                    <button onClick={handleGenerateAnalysis} disabled={isAnalyzing} className="mt-4 bg-indigo-600 text-white px-4 py-2 rounded-lg shadow hover:bg-indigo-700 transition-all text-sm font-semibold flex items-center justify-center disabled:opacity-50">
                        {isAnalyzing ? 'Menganalisis...' : <><Sparkles className="w-4 h-4 mr-2" /> Buat Analisis</>}
                    </button>
                    {analysis && (
                        <div className="mt-4 p-4 bg-slate-50 rounded-lg border prose prose-sm max-w-none">
                            {analysis.split('\n').map((line, index) => {
                                if (line.startsWith('* ') || line.startsWith('- ')) {
                                    return <p key={index} className="ml-4">{line}</p>;
                                }
                                return <p key={index}>{line}</p>;
                            })}
                        </div>
                    )}
                </div>
                <div className="bg-white p-6 rounded-xl shadow-md border border-slate-200">
                    <h3 className="text-lg font-semibold text-slate-700">Laporan Anggota</h3>
                    <p className="text-sm text-slate-500 mt-1">Ekspor semua data anggota POKMAS.</p>
                    <button onClick={() => exportToCSV(members, 'laporan_anggota')} className="mt-4 w-full bg-indigo-600 text-white px-4 py-2 rounded-lg shadow hover:bg-indigo-700 transition-all text-sm font-semibold flex items-center justify-center">
                        <Download className="w-4 h-4 mr-2" /> Ekspor ke CSV
                    </button>
                </div>
                <div className="bg-white p-6 rounded-xl shadow-md border border-slate-200">
                    <h3 className="text-lg font-semibold text-slate-700">Laporan Keuangan</h3>
                    <p className="text-sm text-slate-500 mt-1">Ekspor semua riwayat transaksi.</p>
                    <button onClick={() => exportToCSV(transactions, 'laporan_keuangan')} className="mt-4 w-full bg-indigo-600 text-white px-4 py-2 rounded-lg shadow hover:bg-indigo-700 transition-all text-sm font-semibold flex items-center justify-center">
                        <Download className="w-4 h-4 mr-2" /> Ekspor ke CSV
                    </button>
                </div>
                <div className="bg-white p-6 rounded-xl shadow-md border border-slate-200">
                    <h3 className="text-lg font-semibold text-slate-700">Laporan Paket Pekerjaan</h3>
                    <p className="text-sm text-slate-500 mt-1">Ekspor semua data paket pekerjaan.</p>
                    <button onClick={() => exportToCSV(workPackages, 'laporan_paket_pekerjaan')} className="mt-4 w-full bg-indigo-600 text-white px-4 py-2 rounded-lg shadow hover:bg-indigo-700 transition-all text-sm font-semibold flex items-center justify-center">
                        <Download className="w-4 h-4 mr-2" /> Ekspor ke CSV
                    </button>
                </div>
                <div className="bg-white p-6 rounded-xl shadow-md border border-slate-200">
                    <h3 className="text-lg font-semibold text-slate-700">Laporan Tagihan Upah</h3>
                    <p className="text-sm text-slate-500 mt-1">Ekspor semua riwayat tagihan upah.</p>
                    <button onClick={() => exportToCSV(wageBills, 'laporan_tagihan_upah')} className="mt-4 w-full bg-indigo-600 text-white px-4 py-2 rounded-lg shadow hover:bg-indigo-700 transition-all text-sm font-semibold flex items-center justify-center">
                        <Download className="w-4 h-4 mr-2" /> Ekspor ke CSV
                    </button>
                </div>
                <div className="bg-white p-6 rounded-xl shadow-md border border-slate-200">
                    <h3 className="text-lg font-semibold text-slate-700">Laporan Tagihan Material</h3>
                    <p className="text-sm text-slate-500 mt-1">Ekspor semua riwayat tagihan material.</p>
                    <button onClick={() => exportToCSV(materialBills, 'laporan_tagihan_material')} className="mt-4 w-full bg-indigo-600 text-white px-4 py-2 rounded-lg shadow hover:bg-indigo-700 transition-all text-sm font-semibold flex items-center justify-center">
                        <Download className="w-4 h-4 mr-2" /> Ekspor ke CSV
                    </button>
                </div>
            </div>
        </div>
    );
};

const TanyaRabPage = () => {
    const [jobDescription, setJobDescription] = useState('');
    const [generatedRab, setGeneratedRab] = useState('');
    const [isGenerating, setIsGenerating] = useState(false);

    const handleGenerateRab = async () => {
        if (!jobDescription.trim()) {
            alert("Mohon masukkan deskripsi pekerjaan terlebih dahulu.");
            return;
        }
        setIsGenerating(true);
        setGeneratedRab('');
        const prompt = `
            Anda adalah seorang ahli estimasi biaya untuk proyek pembangunan desa di Indonesia. 
            Buatkan draf Rencana Anggaran Biaya (RAB) sederhana dalam format markdown (tabel) untuk pekerjaan berikut: "${jobDescription}".

            Tabel RAB harus mencakup kolom-kolom berikut:
            - No.
            - Uraian Pekerjaan
            - Volume
            - Satuan
            - Harga Satuan (Rp)
            - Jumlah (Rp)

            Sertakan perkiraan untuk biaya material dan upah tenaga kerja. Asumsikan harga standar untuk material dan upah di daerah pedesaan Indonesia. Berikan juga total estimasi biaya di bagian bawah.
            PENTING: Jangan membuat asumsi harga yang terlalu spesifik, berikan rentang atau perkiraan umum. Jawaban harus berupa tabel markdown yang rapi.
        `;
        try {
            const result = await callGeminiApi(prompt);
            setGeneratedRab(result);
        } catch (error) {
            setGeneratedRab("Gagal membuat RAB. Silakan coba lagi.");
        } finally {
            setIsGenerating(false);
        }
    };

    const handleClearChat = () => {
        setJobDescription('');
        setGeneratedRab('');
    };

    return (
        <div>
            <h2 className="text-3xl font-bold text-slate-800">Tanya RAB Pekerjaan</h2>
            <p className="text-slate-500 mt-1">Dapatkan draf estimasi Rencana Anggaran Biaya (RAB) secara otomatis dengan AI.</p>

            <div className="mt-6 bg-white p-6 rounded-xl shadow-md border border-slate-200">
                <div className="space-y-4">
                    <div>
                        <label htmlFor="jobDescription" className="block text-sm font-medium text-slate-700 mb-1">
                            Deskripsikan Pekerjaan
                        </label>
                        <textarea
                            id="jobDescription"
                            rows="4"
                            className="w-full px-3 py-2 border border-slate-300 rounded-lg focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500 transition"
                            placeholder="Contoh: Pembangunan jalan rabat beton sepanjang 100 meter, lebar 3 meter, dan tebal 15 cm di Desa Maju Jaya."
                            value={jobDescription}
                            onChange={(e) => setJobDescription(e.target.value)}
                        ></textarea>
                    </div>
                    <div className="flex space-x-2">
                        <button 
                            onClick={handleGenerateRab} 
                            disabled={isGenerating} 
                            className="bg-indigo-600 text-white px-4 py-2 rounded-lg shadow hover:bg-indigo-700 transition-all text-sm font-semibold flex items-center justify-center disabled:opacity-50"
                        >
                            {isGenerating ? 'Membuat Draf...' : <><Sparkles className="w-4 h-4 mr-2" /> Buat Draf RAB</>}
                        </button>
                        {(jobDescription || generatedRab) && (
                             <button 
                                onClick={handleClearChat} 
                                className="bg-slate-200 text-slate-700 px-4 py-2 rounded-lg hover:bg-slate-300 transition-all text-sm font-semibold flex items-center justify-center"
                            >
                                <XCircle className="w-4 h-4 mr-2" /> Bersihkan
                            </button>
                        )}
                    </div>
                </div>

                {generatedRab && (
                    <div className="mt-6">
                        <h3 className="text-lg font-semibold text-slate-700">Hasil Draf RAB</h3>
                        <div className="mt-2 p-4 bg-slate-50 rounded-lg border prose prose-sm max-w-none prose-p:my-1 prose-table:my-2">
                            {generatedRab.split('\n').map((line, index) => {
                                // Simple markdown to HTML conversion for tables
                                if (line.includes('|')) {
                                    const cells = line.split('|').filter(cell => cell.trim() !== '');
                                    if (line.includes('---')) {
                                        return null; // Skip separator line in direct rendering
                                    }
                                    return (
                                        <div key={index} className="flex border-b">
                                            {cells.map((cell, cellIndex) => (
                                                <div key={cellIndex} className="p-2 flex-1">{cell.trim()}</div>
                                            ))}
                                        </div>
                                    );
                                }
                                return <p key={index}>{line}</p>;
                            })}
                        </div>
                    </div>
                )}
            </div>
        </div>
    );
};


// --- KOMPONEN MODAL ---
const ConfirmationModal = ({ closeModal, onConfirm, title, message }) => (
    <div className="p-6">
        <h3 className="text-xl font-bold text-slate-800">{title}</h3>
        <p className="text-slate-500 my-4">{message}</p>
        <div className="mt-6 flex justify-end space-x-3">
            <button type="button" onClick={closeModal} className="px-4 py-2 text-sm font-medium text-slate-700 bg-slate-100 rounded-lg hover:bg-slate-200">Batal</button>
            <button type="button" onClick={onConfirm} className="px-4 py-2 text-sm font-medium text-white bg-red-600 rounded-lg hover:bg-red-700">Ya, Hapus</button>
        </div>
    </div>
);

const TambahTransaksiModal = ({ closeModal, workPackages, transactions, handleAddOrUpdate, dataToEdit }) => {
    const isEditMode = Boolean(dataToEdit);
    const [mode, setMode] = useState(isEditMode && dataToEdit.kategori.includes('Pencairan Dana') ? 'pencairan' : 'umum');
    const [selectedPackage, setSelectedPackage] = useState(isEditMode ? dataToEdit.paketPekerjaan : '');
    const [selectedStage, setSelectedStage] = useState('');
    const [error, setError] = useState('');

    const [formData, setFormData] = useState(
        isEditMode ? dataToEdit : {
            tanggal: new Date().toISOString().split('T')[0],
            jenis: 'Pengeluaran',
            kategori: '',
            jumlah: 0,
            paketPekerjaan: '',
            keterangan: '',
            status: 'Cair',
        }
    );

    const handleSubmit = (e) => {
        e.preventDefault();
        const dataToSubmit = { ...formData, jumlah: Number(formData.jumlah) };
        handleAddOrUpdate('transaksi', dataToSubmit, dataToEdit?.id);
    };

    const handleChange = (e) => {
        const { id, value } = e.target;
        setFormData(prev => ({ ...prev, [id]: value }));
    };
    
    useEffect(() => {
        if (mode === 'pencairan' && !isEditMode && selectedPackage && selectedStage) {
            const workPackage = workPackages.find(wp => wp.namaPaket === selectedPackage);
            if (!workPackage) {
                setError('Paket pekerjaan tidak ditemukan.');
                return;
            }

            const isAlreadyDisbursed = transactions.some(t => 
                t.paketPekerjaan === selectedPackage && 
                t.kategori.includes(selectedStage) &&
                t.status === 'Cair'
            );

            if (isAlreadyDisbursed) {
                setError(`Dana untuk ${selectedStage} pada paket ini sudah dicairkan.`);
                setFormData(prev => ({ ...prev, jumlah: 0, keterangan: '' }));
                return;
            }

            const stages = { 'Tahap 1': 0.4, 'Tahap 2': 0.3, 'Tahap 3': 0.3 };
            const percentage = stages[selectedStage];
            const calculatedAmount = (workPackage.nilaiPagu || 0) * percentage;
            
            setError('');
            setFormData(prev => ({
                ...prev,
                jenis: 'Pemasukan',
                kategori: `Pencairan Dana ${selectedStage}`,
                jumlah: calculatedAmount,
                paketPekerjaan: selectedPackage,
                keterangan: `Pencairan dana untuk ${selectedStage} (${percentage * 100}%) paket: ${selectedPackage}`,
                status: 'Cair'
            }));
        }
    }, [mode, selectedPackage, selectedStage, transactions, workPackages, isEditMode]);


    return (
        <div>
            <div className="flex justify-between items-center p-4 border-b border-slate-200">
                <h3 className="text-xl font-bold text-slate-800">{isEditMode ? 'Edit Transaksi' : 'Tambah Transaksi'}</h3>
                <button onClick={closeModal} className="text-slate-400 hover:text-slate-600"><X className="w-6 h-6" /></button>
            </div>
            <div className="p-6">
                {!isEditMode && (
                    <div className="mb-6"><div className="flex space-x-4 border-b pb-4"><label className="flex items-center"><input type="radio" name="mode" value="umum" checked={mode === 'umum'} onChange={(e) => setMode(e.target.value)} className="h-4 w-4"/> <span className="ml-2 text-sm">Transaksi Umum</span></label><label className="flex items-center"><input type="radio" name="mode" value="pencairan" checked={mode === 'pencairan'} onChange={(e) => setMode(e.target.value)} className="h-4 w-4"/> <span className="ml-2 text-sm">Pencairan Dana Paket</span></label></div></div>
                )}

                <form onSubmit={handleSubmit}>
                    {mode === 'pencairan' && !isEditMode ? (
                        <div className="space-y-4">
                            <FormSelect label="Paket Pekerjaan" value={selectedPackage} onChange={(e) => setSelectedPackage(e.target.value)}>
                                <option key="default-paket-transaksi" value="">-- Pilih Paket --</option>
                                {workPackages.map(wp => (<option key={wp.id} value={wp.namaPaket}>{wp.namaPaket}</option>))}
                            </FormSelect>
                            <FormSelect label="Tahap Pencairan" value={selectedStage} onChange={(e) => setSelectedStage(e.target.value)} disabled={!selectedPackage}>
                                <option key="pilih" value="">-- Pilih Tahap --</option>
                                <option key="t1" value="Tahap 1">Tahap 1 (40%)</option>
                                <option key="t2" value="Tahap 2">Tahap 2 (30%)</option>
                                <option key="t3" value="Tahap 3">Tahap 3 (30%)</option>
                            </FormSelect>
                            {error && <p className="text-sm text-red-600">{error}</p>}
                            <FormInput label="Jumlah (Rp)" id="jumlah" type="number" value={formData.jumlah} readOnly className="bg-slate-100" />
                            <FormInput label="Keterangan" id="keterangan" type="text" value={formData.keterangan} readOnly className="bg-slate-100" />
                        </div>
                    ) : (
                        <div className="space-y-4">
                            <FormInput label="Tanggal" id="tanggal" type="date" value={formData.tanggal} onChange={handleChange} />
                            <FormSelect label="Jenis" id="jenis" value={formData.jenis} onChange={handleChange}>
                                <option key="pengeluaran" value="Pengeluaran">Pengeluaran</option>
                                <option key="pemasukan" value="Pemasukan">Pemasukan</option>
                            </FormSelect>
                            <FormInput label="Kategori" id="kategori" type="text" value={formData.kategori} onChange={handleChange} placeholder="e.g., Biaya Konsumsi, Pembelian ATK" />
                            <FormInput label="Jumlah (Rp)" id="jumlah" type="number" value={formData.jumlah} onChange={handleChange} />
                            <FormSelect label="Terkait Paket Pekerjaan (Opsional)" id="paketPekerjaan" value={formData.paketPekerjaan} onChange={handleChange}><option key="default-paket-opsional" value="">-- Tidak terikat --</option>{workPackages.map(wp => (<option key={wp.id} value={wp.namaPaket}>{wp.namaPaket}</option>))}</FormSelect>
                            <FormInput label="Keterangan" id="keterangan" type="text" value={formData.keterangan} onChange={handleChange} />
                        </div>
                    )}
                    <div className="mt-8 flex justify-end">
                        <button type="button" onClick={closeModal} className="px-4 py-2 text-sm font-medium text-slate-700 bg-slate-100 rounded-lg hover:bg-slate-200 mr-2">Batal</button>
                        <button type="submit" disabled={mode === 'pencairan' && !!error} className="px-4 py-2 text-sm font-medium text-white bg-indigo-600 rounded-lg hover:bg-indigo-700 disabled:opacity-50 disabled:cursor-not-allowed">
                            {isEditMode ? 'Simpan Perubahan' : (mode === 'pencairan' ? 'Cairkan Dana' : 'Simpan Transaksi')}
                        </button>
                    </div>
                </form>
            </div>
        </div>
    );
};

const TambahAnggotaModal = ({ closeModal, dataToEdit, handleAddOrUpdate }) => {
    const isEditMode = Boolean(dataToEdit);
    const [formData, setFormData] = useState({
        namaLengkap: dataToEdit?.namaLengkap || '',
        jabatan: dataToEdit?.jabatan || '',
        nomorTelepon: dataToEdit?.nomorTelepon || '',
    });

    const handleChange = (e) => {
        const { id, value } = e.target;
        setFormData(prev => ({ ...prev, [id]: value }));
    };

    const handleSubmit = (e) => {
        e.preventDefault();
        handleAddOrUpdate('anggota', formData, dataToEdit?.id);
    };

    return (
        <div>
            <div className="flex justify-between items-center p-4 border-b border-slate-200">
                <h3 className="text-xl font-bold text-slate-800">{isEditMode ? 'Edit Anggota' : 'Tambah Anggota Baru'}</h3>
                <button onClick={closeModal} className="text-slate-400 hover:text-slate-600"><X className="w-6 h-6" /></button>
            </div>
            <form onSubmit={handleSubmit} className="p-6">
                <div className="space-y-4">
                    <FormInput label="Nama Lengkap" id="namaLengkap" type="text" value={formData.namaLengkap} onChange={handleChange} />
                    <FormInput label="Jabatan" id="jabatan" type="text" value={formData.jabatan} onChange={handleChange} />
                    <FormInput label="Nomor Telepon" id="nomorTelepon" type="tel" value={formData.nomorTelepon} onChange={handleChange} />
                </div>
                <div className="mt-8 flex justify-end"><button type="button" onClick={closeModal} className="px-4 py-2 text-sm font-medium text-slate-700 bg-slate-100 rounded-lg hover:bg-slate-200 mr-2">Batal</button><button type="submit" className="px-4 py-2 text-sm font-medium text-white bg-indigo-600 rounded-lg hover:bg-indigo-700">{isEditMode ? 'Simpan Perubahan' : 'Simpan'}</button></div>
            </form>
        </div>
    );
};

const TambahPaketPekerjaanModal = ({ closeModal, dataToEdit, handleAddOrUpdate }) => {
    const isEditMode = Boolean(dataToEdit);
    const [file, setFile] = useState(null);
    const [isGeneratingDesc, setIsGeneratingDesc] = useState(false);
    const [formData, setFormData] = useState({
        namaPaket: dataToEdit?.namaPaket || '',
        lokasi: dataToEdit?.lokasi || '',
        volume: dataToEdit?.volume || '',
        nilaiPagu: dataToEdit?.nilaiPagu || 0,
        deskripsi: dataToEdit?.deskripsi || '',
        tanggalMulai: dataToEdit?.tanggalMulai || new Date().toISOString().split('T')[0],
        tanggalSelesai: dataToEdit?.tanggalSelesai || '',
        status: dataToEdit?.status || 'Direncanakan',
    });
    
    const handleChange = (e) => {
        const { id, value } = e.target;
        setFormData(prev => ({ ...prev, [id]: value }));
    };

    const handleFileChange = (e) => {
        setFile(e.target.files[0]);
    };

    const handleSubmit = (e) => {
        e.preventDefault();
        const dataToSubmit = { ...formData, nilaiPagu: Number(formData.nilaiPagu) };
        handleAddOrUpdate('paket', dataToSubmit, dataToEdit?.id, file);
    };

    const handleGenerateDescription = async () => {
        if (!formData.namaPaket || !formData.lokasi) {
            alert("Mohon isi Nama Paket dan Lokasi terlebih dahulu.");
            return;
        }
        setIsGeneratingDesc(true);
        const prompt = `Buatkan deskripsi singkat (sekitar 2-3 kalimat) untuk proyek pembangunan masyarakat bernama "${formData.namaPaket}" yang berlokasi di "${formData.lokasi}". Jelaskan tujuan umum dari proyek tersebut.`;
        try {
            const result = await callGeminiApi(prompt);
            setFormData(prev => ({ ...prev, deskripsi: result }));
        } catch (error) {
            alert("Gagal membuat deskripsi. Silakan coba lagi.");
        } finally {
            setIsGeneratingDesc(false);
        }
    };

    return (
        <div>
            <div className="flex justify-between items-center p-4 border-b border-slate-200">
                <h3 className="text-xl font-bold text-slate-800">{isEditMode ? 'Edit Paket Pekerjaan' : 'Tambah Paket Pekerjaan'}</h3>
                <button onClick={closeModal} className="text-slate-400 hover:text-slate-600"><X className="w-6 h-6" /></button>
            </div>
            <form onSubmit={handleSubmit} className="p-6">
                <div className="space-y-4">
                    <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
                        <FormInput label="Nama Paket" id="namaPaket" type="text" value={formData.namaPaket} onChange={handleChange} />
                        <FormInput label="Lokasi" id="lokasi" type="text" value={formData.lokasi} onChange={handleChange} />
                    </div>
                    <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
                        <FormInput label="Volume" id="volume" type="text" value={formData.volume} onChange={handleChange} />
                        <FormInput label="Nilai Pagu Total (Rp)" id="nilaiPagu" type="number" value={formData.nilaiPagu} onChange={handleChange} />
                    </div>
                    <div>
                        <div className="flex justify-between items-center mb-1">
                           <label htmlFor="deskripsi" className="block text-sm font-medium text-slate-700">Deskripsi</label>
                           <button type="button" onClick={handleGenerateDescription} disabled={isGeneratingDesc} className="text-xs bg-indigo-100 text-indigo-700 font-semibold py-1 px-2 rounded-full flex items-center disabled:opacity-50">
                               <Sparkles className="w-3 h-3 mr-1" /> {isGeneratingDesc ? 'Membuat...' : 'Buat Otomatis'}
                           </button>
                        </div>
                        <textarea id="deskripsi" rows="3" className="w-full px-3 py-2 border border-slate-300 rounded-lg" value={formData.deskripsi} onChange={handleChange}></textarea>
                    </div>
                     <div>
                        <label htmlFor="file" className="block text-sm font-medium text-slate-700 mb-1">File Lampiran (Excel, PDF)</label>
                        <input type="file" id="file" onChange={handleFileChange} className="block w-full text-sm text-slate-500 file:mr-4 file:py-2 file:px-4 file:rounded-full file:border-0 file:text-sm file:font-semibold file:bg-indigo-50 file:text-indigo-700 hover:file:bg-indigo-100"/>
                        {file && <span className="text-xs text-slate-500 mt-1">{file.name}</span>}
                        {isEditMode && dataToEdit?.fileName && <span className="text-xs text-slate-500 mt-1">File saat ini: {dataToEdit.fileName}</span>}
                    </div>
                    <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
                        <FormInput label="Tanggal Mulai" id="tanggalMulai" type="date" value={formData.tanggalMulai} onChange={handleChange} />
                        <FormInput label="Tanggal Selesai" id="tanggalSelesai" type="date" value={formData.tanggalSelesai} onChange={handleChange} />
                    </div>
                    <FormSelect label="Status" id="status" value={formData.status} onChange={handleChange}>
                        <option key="direncanakan" value="Direncanakan">Direncanakan</option>
                        <option key="berjalan" value="Berjalan">Berjalan</option>
                        <option key="selesai" value="Selesai">Selesai</option>
                    </FormSelect>
                </div>
                <div className="mt-8 flex justify-end"><button type="button" onClick={closeModal} className="px-4 py-2 text-sm font-medium text-slate-700 bg-slate-100 rounded-lg hover:bg-slate-200 mr-2">Batal</button><button type="submit" className="px-4 py-2 text-sm font-medium text-white bg-indigo-600 rounded-lg hover:bg-indigo-700">{isEditMode ? 'Simpan Perubahan' : 'Simpan'}</button></div>
            </form>
        </div>
    );
};

const TambahTagihanUpahModal = ({ closeModal, dataToEdit, handleAddOrUpdate, workPackages }) => {
    const isEditMode = Boolean(dataToEdit);
    const [formData, setFormData] = useState({
        namaPekerja: dataToEdit?.namaPekerja || '',
        paketPekerjaan: dataToEdit?.paketPekerjaan || '',
        kategori: dataToEdit?.kategori || 'Cashbon',
        jumlah: dataToEdit?.jumlah || 0,
        tanggal: dataToEdit?.tanggal || new Date().toISOString().split('T')[0],
        status: dataToEdit?.status || 'Belum Dibayar',
        keterangan: dataToEdit?.keterangan || '',
    });
    
    const handleChange = (e) => {
        const { id, value } = e.target;
        setFormData(prev => ({ ...prev, [id]: value }));
    };

    const handleSubmit = (e) => {
        e.preventDefault();
        const dataToSubmit = { ...formData, jumlah: Number(formData.jumlah) };
        handleAddOrUpdate('tagihan-upah', dataToSubmit, dataToEdit?.id);
    };

    return (
        <div>
            <div className="flex justify-between items-center p-4 border-b border-slate-200">
                <h3 className="text-xl font-bold text-slate-800">{isEditMode ? 'Edit Tagihan Upah' : 'Tambah Tagihan Upah'}</h3>
                <button onClick={closeModal} className="text-slate-400 hover:text-slate-600"><X className="w-6 h-6" /></button>
            </div>
            <form onSubmit={handleSubmit} className="p-6">
                <div className="space-y-4">
                    <FormInput label="Nama Pekerja" id="namaPekerja" type="text" value={formData.namaPekerja} onChange={handleChange} />
                    <FormSelect label="Paket Pekerjaan" id="paketPekerjaan" value={formData.paketPekerjaan} onChange={handleChange}>
                        <option key="default-paket-upah" value="">-- Pilih Paket --</option>
                        {workPackages.map(wp => (<option key={wp.id} value={wp.namaPaket}>{wp.namaPaket}</option>))}
                    </FormSelect>
                    <FormSelect label="Kategori Pembayaran" id="kategori" value={formData.kategori} onChange={handleChange}>
                        <option key="cashbon" value="Cashbon">Cashbon</option>
                        <option key="opname" value="Cash sesuai opname">Cash sesuai opname</option>
                        <option key="volume" value="Cash sesuai volume">Cash sesuai volume</option>
                    </FormSelect>
                    <FormInput label="Jumlah Tagihan (Rp)" id="jumlah" type="number" value={formData.jumlah} onChange={handleChange} />
                    <FormInput label="Tanggal" id="tanggal" type="date" value={formData.tanggal} onChange={handleChange} />
                    <FormSelect label="Status" id="status" value={formData.status} onChange={handleChange}>
                        <option key="belum" value="Belum Dibayar">Belum Dibayar</option>
                        <option key="lunas" value="Lunas">Lunas</option>
                    </FormSelect>
                    <FormInput label="Keterangan" id="keterangan" type="text" value={formData.keterangan} onChange={handleChange} />
                </div>
                <div className="mt-8 flex justify-end"><button type="button" onClick={closeModal} className="px-4 py-2 text-sm font-medium text-slate-700 bg-slate-100 rounded-lg hover:bg-slate-200 mr-2">Batal</button><button type="submit" className="px-4 py-2 text-sm font-medium text-white bg-indigo-600 rounded-lg hover:bg-indigo-700">{isEditMode ? 'Simpan Perubahan' : 'Simpan'}</button></div>
            </form>
        </div>
    );
};

const TambahTagihanMaterialModal = ({ closeModal, dataToEdit, handleAddOrUpdate, workPackages }) => {
    const isEditMode = Boolean(dataToEdit);
    const [formData, setFormData] = useState({
        toko: dataToEdit?.toko || '',
        uraian: dataToEdit?.uraian || '',
        volume: dataToEdit?.volume || 0,
        hargaSatuan: dataToEdit?.hargaSatuan || 0,
        total: dataToEdit?.total || 0,
        tanggal: dataToEdit?.tanggal || new Date().toISOString().split('T')[0],
        status: dataToEdit?.status || 'Belum Lunas',
        paketPekerjaan: dataToEdit?.paketPekerjaan || '',
    });
    
    useEffect(() => {
        const newTotal = formData.volume * formData.hargaSatuan;
        setFormData(prev => ({ ...prev, total: newTotal }));
    }, [formData.volume, formData.hargaSatuan]);

    const handleChange = (e) => {
        const { id, value } = e.target;
        setFormData(prev => ({ ...prev, [id]: value }));
    };

    const handleSubmit = (e) => {
        e.preventDefault();
        const dataToSubmit = { 
            ...formData, 
            volume: Number(formData.volume),
            hargaSatuan: Number(formData.hargaSatuan),
            total: Number(formData.total),
        };
        handleAddOrUpdate('tagihan-material', dataToSubmit, dataToEdit?.id);
    };

    return (
        <div>
            <div className="flex justify-between items-center p-4 border-b border-slate-200">
                <h3 className="text-xl font-bold text-slate-800">{isEditMode ? 'Edit Tagihan Material' : 'Tambah Tagihan Material'}</h3>
                <button onClick={closeModal} className="text-slate-400 hover:text-slate-600"><X className="w-6 h-6" /></button>
            </div>
            <form onSubmit={handleSubmit} className="p-6">
                <div className="space-y-4">
                    <FormInput label="Nama Toko" id="toko" type="text" value={formData.toko} onChange={handleChange} />
                    <FormInput label="Uraian (Nama Barang)" id="uraian" type="text" value={formData.uraian} onChange={handleChange} />
                    <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
                        <FormInput label="Volume" id="volume" type="number" value={formData.volume} onChange={handleChange} />
                        <FormInput label="Harga Satuan (Rp)" id="hargaSatuan" type="number" value={formData.hargaSatuan} onChange={handleChange} />
                    </div>
                    <FormInput label="Total (Rp)" id="total" type="number" value={formData.total} readOnly className="bg-slate-100" />
                    <FormSelect label="Paket Pekerjaan" id="paketPekerjaan" value={formData.paketPekerjaan} onChange={handleChange}><option key="default-paket-material" value="">-- Pilih Paket --</option>{workPackages.map(wp => (<option key={wp.id} value={wp.namaPaket}>{wp.namaPaket}</option>))}</FormSelect>
                    <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
                        <FormInput label="Tanggal" id="tanggal" type="date" value={formData.tanggal} onChange={handleChange} />
                        <FormSelect label="Status" id="status" value={formData.status} onChange={handleChange}>
                            <option key="belum" value="Belum Lunas">Belum Lunas</option>
                            <option key="lunas" value="Lunas">Lunas</option>
                        </FormSelect>
                    </div>
                </div>
                <div className="mt-8 flex justify-end"><button type="button" onClick={closeModal} className="px-4 py-2 text-sm font-medium text-slate-700 bg-slate-100 rounded-lg hover:bg-slate-200 mr-2">Batal</button><button type="submit" className="px-4 py-2 text-sm font-medium text-white bg-indigo-600 rounded-lg hover:bg-indigo-700">{isEditMode ? 'Simpan Perubahan' : 'Simpan'}</button></div>
            </form>
        </div>
    );
};


// --- KOMPONEN BANTUAN ---
const FormInput = ({ label, id, ...props }) => (
    <div><label htmlFor={id} className="block text-sm font-medium text-slate-700 mb-1">{label}</label><input id={id} {...props} className="w-full px-3 py-2 border border-slate-300 rounded-lg focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500 transition" /></div>
);

const FormSelect = ({ label, id, children, ...props }) => (
    <div><label htmlFor={id} className="block text-sm font-medium text-slate-700 mb-1">{label}</label><select id={id} {...props} className="w-full px-3 py-2 border border-slate-300 rounded-lg focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500 transition bg-white">{children}</select></div>
);

const MultiSelectDropdown = ({ options, selectedOptions, onChange, label }) => {
    const [isOpen, setIsOpen] = useState(false);
    const dropdownRef = useRef(null);

    const handleToggle = (option) => {
        if (selectedOptions.includes(option)) {
            onChange(selectedOptions.filter(item => item !== option));
        } else {
            onChange([...selectedOptions, option]);
        }
    };

    useEffect(() => {
        const handleClickOutside = (event) => {
            if (dropdownRef.current && !dropdownRef.current.contains(event.target)) {
                setIsOpen(false);
            }
        };
        document.addEventListener("mousedown", handleClickOutside);
        return () => document.removeEventListener("mousedown", handleClickOutside);
    }, [dropdownRef]);

    return (
        <div className="relative" ref={dropdownRef}>
            <label className="block text-sm font-medium text-slate-700 mb-1">{label}</label>
            <button type="button" onClick={() => setIsOpen(!isOpen)} className="w-full px-3 py-2 border border-slate-300 rounded-lg bg-white text-left flex justify-between items-center">
                <span className="text-slate-700">
                    {selectedOptions.length > 0 ? `${selectedOptions.length} paket dipilih` : "Pilih paket pekerjaan..."}
                </span>
                <ChevronDown className={`w-5 h-5 text-slate-400 transition-transform ${isOpen ? 'transform rotate-180' : ''}`} />
            </button>
            {isOpen && (
                <div className="absolute z-10 w-full mt-1 bg-white border border-slate-300 rounded-lg shadow-lg max-h-60 overflow-y-auto">
                    {options.map(option => (
                        <label key={option.id} className="flex items-center px-4 py-2 text-sm text-slate-700 hover:bg-slate-100 cursor-pointer">
                            <input type="checkbox" checked={selectedOptions.includes(option.namaPaket)} onChange={() => handleToggle(option.namaPaket)} className="h-4 w-4 rounded border-slate-300 text-indigo-600 focus:ring-indigo-500" />
                            <span className="ml-3">{option.namaPaket}</span>
                        </label>
                    ))}
                </div>
            )}
        </div>
    );
};

const TambahRincianTagihanForm = ({ workPackages, handleAddOrUpdate, setActiveTab }) => {
    const [selectedPackages, setSelectedPackages] = useState([]);
    const [namaPekerja, setNamaPekerja] = useState('');
    const [nilai, setNilai] = useState(0);
    const [cashBon, setCashBon] = useState(0);
    const sisaUang = nilai - cashBon;

    const handleSubmit = (e) => {
        e.preventDefault();
        if (selectedPackages.length === 0 || !namaPekerja || nilai <= 0) {
            alert("Mohon lengkapi Nama Pekerja, Paket Pekerjaan, dan Nilai.");
            return;
        }

        const newBill = {
            namaPekerja,
            paketPekerjaan: selectedPackages,
            nilaiTotal: Number(nilai),
            cashBon: Number(cashBon),
            jumlah: Number(sisaUang),
            tanggal: new Date().toISOString().split('T')[0],
            status: 'Belum Dibayar',
            kategori: 'Rincian Proyek Ganda',
            keterangan: `Tagihan untuk paket: ${selectedPackages.join(', ')}`
        };

        addDoc(collection(db, 'wage-bills'), newBill)
            .then(() => {
                alert("Rincian tagihan berhasil disimpan!");
                // Reset form
                setSelectedPackages([]);
                setNamaPekerja('');
                setNilai(0);
                setCashBon(0);
                setActiveTab('Daftar Tagihan');
            })
            .catch(error => {
                console.error("Error adding document: ", error);
                alert("Gagal menyimpan data.");
            });
    };

    return (
        <div className="mt-6 bg-white p-6 rounded-xl shadow-md border border-slate-200">
            <form onSubmit={handleSubmit} className="space-y-4 max-w-lg mx-auto">
                <MultiSelectDropdown 
                    label="Nama Paket Pekerjaan"
                    options={workPackages}
                    selectedOptions={selectedPackages}
                    onChange={setSelectedPackages}
                />
                <FormInput label="Nama Pekerja" id="namaPekerja" type="text" value={namaPekerja} onChange={(e) => setNamaPekerja(e.target.value)} required />
                <FormInput label="Nilai (Rp)" id="nilai" type="number" value={nilai} onChange={(e) => setNilai(e.target.value)} required />
                <FormInput label="Cash Bon (Rp)" id="cashBon" type="number" value={cashBon} onChange={(e) => setCashBon(e.target.value)} />
                <div>
                    <label className="block text-sm font-medium text-slate-700 mb-1">Sisa Uang (Rp)</label>
                    <div className="w-full px-3 py-2 bg-slate-100 border border-slate-300 rounded-lg">
                        {new Intl.NumberFormat('id-ID').format(sisaUang)}
                    </div>
                </div>
                <div className="pt-4">
                    <button type="submit" className="w-full bg-indigo-600 text-white px-4 py-3 rounded-lg shadow hover:bg-indigo-700 transition-all font-semibold flex items-center justify-center">
                        <Plus className="w-5 h-5 mr-2" /> Simpan Tagihan
                    </button>
                </div>
            </form>
        </div>
    );
};

const UploadFileForPackageModal = ({ closeModal, workPackages, handleAddOrUpdate }) => {
    const [selectedPackageId, setSelectedPackageId] = useState('');
    const [file, setFile] = useState(null);

    const handleSubmit = (e) => {
        e.preventDefault();
        if (!selectedPackageId || !file) {
            alert("Silakan pilih paket pekerjaan dan file terlebih dahulu.");
            return;
        }
        // Kirim data kosong {} karena kita hanya ingin memperbarui file
        handleAddOrUpdate('paket', {}, selectedPackageId, file);
    };

    return (
        <div>
            <div className="flex justify-between items-center p-4 border-b border-slate-200">
                <h3 className="text-xl font-bold text-slate-800">Upload File Lampiran</h3>
                <button onClick={closeModal} className="text-slate-400 hover:text-slate-600"><X className="w-6 h-6" /></button>
            </div>
            <form onSubmit={handleSubmit} className="p-6">
                <div className="space-y-4">
                    <FormSelect label="Pilih Paket Pekerjaan" value={selectedPackageId} onChange={(e) => setSelectedPackageId(e.target.value)}>
                        <option key="default-paket-upload" value="">-- Pilih Paket --</option>
                        {workPackages.map(wp => (<option key={wp.id} value={wp.id}>{wp.namaPaket}</option>))}
                    </FormSelect>
                    <div>
                        <label htmlFor="fileUpload" className="block text-sm font-medium text-slate-700 mb-1">Pilih File (Excel, PDF)</label>
                        <input type="file" id="fileUpload" onChange={(e) => setFile(e.target.files[0])} className="block w-full text-sm text-slate-500 file:mr-4 file:py-2 file:px-4 file:rounded-full file:border-0 file:text-sm file:font-semibold file:bg-indigo-50 file:text-indigo-700 hover:file:bg-indigo-100"/>
                        {file && <span className="text-xs text-slate-500 mt-1">{file.name}</span>}
                    </div>
                </div>
                <div className="mt-8 flex justify-end">
                    <button type="button" onClick={closeModal} className="px-4 py-2 text-sm font-medium text-slate-700 bg-slate-100 rounded-lg hover:bg-slate-200 mr-2">Batal</button>
                    <button type="submit" className="px-4 py-2 text-sm font-medium text-white bg-indigo-600 rounded-lg hover:bg-indigo-700">Upload File</button>
                </div>
            </form>
        </div>
    );
};

const UploadModal = ({ closeModal, type, handleBulkAdd }) => {
    const [file, setFile] = useState(null);
    const [parsedData, setParsedData] = useState([]);
    const [headers, setHeaders] = useState([]);

    const handleFileChange = (e) => {
        const selectedFile = e.target.files[0];
        if (selectedFile && selectedFile.type === 'text/csv') {
            setFile(selectedFile);
            const reader = new FileReader();
            reader.onload = (event) => {
                const text = event.target.result;
                const rows = text.split('\n').filter(row => row.trim() !== '');
                const headerRow = rows[0].split(',');
                setHeaders(headerRow);
                const data = rows.slice(1).map(row => {
                    const values = row.split(',');
                    return headerRow.reduce((obj, header, index) => {
                        obj[header.trim()] = values[index].trim();
                        return obj;
                    }, {});
                });
                setParsedData(data);
            };
            reader.readAsText(selectedFile);
        } else {
            alert("Silakan pilih file dengan format .csv");
        }
    };

    const handleImport = () => {
        handleBulkAdd(type, parsedData);
    };

    return (
        <div>
            <div className="flex justify-between items-center p-4 border-b border-slate-200">
                <h3 className="text-xl font-bold text-slate-800">Upload File CSV</h3>
                <button onClick={closeModal} className="text-slate-400 hover:text-slate-600"><X className="w-6 h-6" /></button>
            </div>
            <div className="p-6">
                <p className="text-sm text-slate-500 mt-2">Pastikan file Excel Anda disimpan sebagai format .csv dengan header yang sesuai.</p>
                <div className="mt-4">
                    <input type="file" accept=".csv" onChange={handleFileChange} className="block w-full text-sm text-slate-500 file:mr-4 file:py-2 file:px-4 file:rounded-full file:border-0 file:text-sm file:font-semibold file:bg-indigo-50 file:text-indigo-700 hover:file:bg-indigo-100"/>
                </div>
                {parsedData.length > 0 && (
                    <div className="mt-4">
                        <h4 className="font-semibold">Pratinjau Data:</h4>
                        <div className="overflow-y-auto h-40 border rounded-lg mt-2">
                            <table className="w-full text-sm">
                                <thead className="bg-slate-50 sticky top-0">
                                    <tr>
                                        {headers.map((h, i) => <th key={`${h}-${i}`} className="px-2 py-1 text-left">{h}</th>)}
                                    </tr>
                                </thead>
                                <tbody>
                                    {parsedData.slice(0, 10).map((row, i) => (
                                        <tr key={i} className="border-b">
                                            {headers.map((h, j) => <td key={`${h}-${j}`} className="px-2 py-1">{row[h]}</td>)}
                                        </tr>
                                    ))}
                                </tbody>
                            </table>
                        </div>
                    </div>
                )}
                <div className="mt-6 flex justify-end space-x-3">
                    <button type="button" onClick={closeModal} className="px-4 py-2 text-sm font-medium text-slate-700 bg-slate-100 rounded-lg hover:bg-slate-200">Batal</button>
                    <button type="button" onClick={handleImport} disabled={parsedData.length === 0} className="px-4 py-2 text-sm font-medium text-white bg-indigo-600 rounded-lg hover:bg-indigo-700 disabled:bg-gray-400">Impor Data</button>
                </div>
            </div>
        </div>
    );
};

export default App;
