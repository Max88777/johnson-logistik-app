const express = require('express');
const cors = require('cors');
const path = require('path');

const app = express();
// Nutze den Port des Cloud-Anbieters oder lokal Port 3001
const PORT = process.env.PORT || 3001;

// Erlaube Verbindungen von anderen Geräten (CORS)
app.use(cors());
app.use(express.json());

// --- ONLINE HOSTING SETUP ---
// Erlaubt es dem Server, die fertige Webseite direkt auszuliefern
app.use(express.static(path.join(__dirname, 'dist')));

// Das ist unsere lokale Datenbank. 
// Sie startet mit einem Standard-Admin-Account.
let db = {
    system_users: [
        { id: '1', username: 'admin', password: '123', role: 'Leitung', createdBy: 'System' }
    ],
    akten: [],
    routes: [],
    shifts: [],
    expenses: [],
    notifications: [],
    emergencies: []
};

// 1. Kompletten Datensatz abrufen (für die Live-Synchronisation der Bildschirme)
app.get('/api/sync', (req, res) => {
    res.json(db);
});

// 2. Ein neues Element hinzufügen (Akte, Route, User, etc.)
app.post('/api/collection/:name', (req, res) => {
    const { name } = req.params;
    if (!db[name]) db[name] = [];
    
    const newItem = { id: Date.now().toString(), ...req.body };
    db[name].push(newItem);
    
    res.json({ success: true, item: newItem });
});

// 3. Ein Element aktualisieren (z.B. Route auf "Abgeschlossen" setzen)
app.put('/api/collection/:name/:id', (req, res) => {
    const { name, id } = req.params;
    if (!db[name]) return res.status(404).json({ error: 'Collection not found' });

    const index = db[name].findIndex(item => item.id === id);
    if (index !== -1) {
        db[name][index] = { ...db[name][index], ...req.body };
        res.json({ success: true, item: db[name][index] });
    } else {
        res.status(404).json({ error: 'Item not found' });
    }
});

// 4. Ein Element löschen (z.B. aus dem Dienst abmelden)
app.delete('/api/collection/:name/:id', (req, res) => {
    const { name, id } = req.params;
    if (!db[name]) return res.status(404).json({ error: 'Collection not found' });

    db[name] = db[name].filter(item => item.id !== id);
    res.json({ success: true });
});

// Fallback für die Online-Version: Alle anderen Routen zeigen die Webseite
app.get('*', (req, res) => {
    res.sendFile(path.join(__dirname, 'dist', 'index.html'));
});

app.listen(PORT, '0.0.0.0', () => {
    console.log(`===========================================`);
    console.log(`Johnson Logistics Zentralserver ist ONLINE!`);
    console.log(`Port: ${PORT}`);
    console.log(`Wartet auf Verbindungen von anderen PCs...`);
    console.log(`===========================================`);
});
