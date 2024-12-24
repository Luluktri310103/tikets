### SQL ###
```sql
-- phpMyAdmin SQL Dump
-- version 5.0.4
-- https://www.phpmyadmin.net/
--
-- Host: 127.0.0.1
-- Waktu pembuatan: 17 Des 2024 pada 06.17
-- Versi server: 10.4.17-MariaDB
-- Versi PHP: 7.3.27

SET SQL_MODE = "NO_AUTO_VALUE_ON_ZERO";
START TRANSACTION;
SET time_zone = "+00:00";


/*!40101 SET @OLD_CHARACTER_SET_CLIENT=@@CHARACTER_SET_CLIENT */;
/*!40101 SET @OLD_CHARACTER_SET_RESULTS=@@CHARACTER_SET_RESULTS */;
/*!40101 SET @OLD_COLLATION_CONNECTION=@@COLLATION_CONNECTION */;
/*!40101 SET NAMES utf8mb4 */;

--
-- Database: `ticketstore`
--

-- --------------------------------------------------------

--
-- Struktur dari tabel `tickets`
--

CREATE TABLE `tickets` (
  `id` int(11) NOT NULL,
  `destination` varchar(30) DEFAULT NULL,
  `date` date DEFAULT NULL,
  `price` decimal(10,2) DEFAULT NULL,
  `stock` int(11) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

--
-- Dumping data untuk tabel `tickets`
--

INSERT INTO `tickets` (`id`, `destination`, `date`, `price`, `stock`) VALUES
(1, 'Jakarta - Kota Metropolitan', '2024-12-10', '550000.00', 90),
(2, 'Bandung - Kota Kembang', '2024-12-11', '320000.00', 140),
(3, 'Surabaya - Kota Pahlawan', '2024-12-12', '480000.00', 75),
(4, 'Yogyakarta - Kota Budaya', '2024-12-13', '360000.00', 110),
(5, 'Bali - Pulau Dewata', '2024-12-14', '620000.00', 55),
(6, 'Medan - Kota Bersejarah', '2024-12-15', '500000.00', 85),
(7, 'Makassar - Kota Pelabuhan', '2024-12-16', '470000.00', 70),
(8, 'Semarang - Kota Pelayaran', '2024-12-17', '400000.00', 95),
(9, 'Palembang - Kota Musi', '2024-12-18', '430000.00', 65),
(10, 'Balikpapan - Kota Minyak', '2024-12-19', '510000.00', 80),
(11, 'Semarang - Purwokerto', '2024-12-20', '250000.00', 70);

--
-- Indexes for dumped tables
--

--
-- Indeks untuk tabel `tickets`
--
ALTER TABLE `tickets`
  ADD PRIMARY KEY (`id`);

--
-- AUTO_INCREMENT untuk tabel yang dibuang
--

--
-- AUTO_INCREMENT untuk tabel `tickets`
--
ALTER TABLE `tickets`
  MODIFY `id` int(11) NOT NULL AUTO_INCREMENT, AUTO_INCREMENT=12;
COMMIT;

/*!40101 SET CHARACTER_SET_CLIENT=@OLD_CHARACTER_SET_CLIENT */;
/*!40101 SET CHARACTER_SET_RESULTS=@OLD_CHARACTER_SET_RESULTS */;
/*!40101 SET COLLATION_CONNECTION=@OLD_COLLATION_CONNECTION */;


```

### Index.html ###
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Daftar Tiket</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.1.3/dist/css/bootstrap.min.css" rel="stylesheet">
    <style>
        .btn-group-action {
            white-space: nowrap;
        }
    </style>
</head>
<body class="container py-4">
    <h1>Daftar Tiket</h1>
    
    <div class="row mb-3">
        <div class="col">
            <input type="text" id="searchInput" class="form-control" placeholder="Cari berdasarkan ID">
        </div>
        <div class="col-auto">
            <button onclick="searchTicket()" class="btn btn-primary">Cari</button>
        </div>
        <div class="col-auto">
            <button type="button" class="btn btn-success" data-bs-toggle="modal" data-bs-target="#ticketModal">
                Tambah Tiket
            </button>
        </div>
    </div>

    <table class="table table-striped">
        <thead class="table-dark">
            <tr>
                <th>ID</th>
                <th>Tujuan</th>
                <th>Tanggal</th>
                <th>Harga</th>
                <th>Stok</th>
                <th>Aksi</th>
            </tr>
        </thead>
        <tbody id="ticketList">
        </tbody>
    </table>

    <!-- Modal untuk Tambah/Edit Tiket -->
    <div class="modal fade" id="ticketModal" tabindex="-1">
        <div class="modal-dialog">
            <div class="modal-content">
                <div class="modal-header">
                    <h5 class="modal-title" id="modalTitle">Tambah Tiket</h5>
                    <button type="button" class="btn-close" data-bs-dismiss="modal"></button>
                </div>
                <div class="modal-body">
                    <form id="ticketForm">
                        <input type="hidden" id="ticketId">
                        <div class="mb-3">
                            <label for="destination" class="form-label">Tujuan</label>
                            <input type="text" class="form-control" id="destination" required>
                        </div>
                        <div class="mb-3">
                            <label for="date" class="form-label">Tanggal</label>
                            <input type="text" class="form-control" id="date" required>
                        </div>
                        <div class="mb-3">
                            <label for="price" class="form-label">Harga</label>
                            <input type="number" class="form-control" id="price" required>
                        </div>
                        <div class="mb-3">
                            <label for="stock" class="form-label">Stok</label>
                            <input type="number" class="form-control" id="stock" required>
                        </div>
                    </form>
                </div>
                <div class="modal-footer">
                    <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">Batal</button>
                    <button type="button" class="btn btn-primary" onclick="saveTicket()">Simpan</button>
                </div>
            </div>
        </div>
    </div>

    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.1.3/dist/js/bootstrap.bundle.min.js"></script>
    <script>
        const API_URL = 'http://localhost/rest_tickets/tickets_api.php';
        let ticketModal;

        document.addEventListener('DOMContentLoaded', function() {
            ticketModal = new bootstrap.Modal(document.getElementById('ticketModal'));
            loadTickets(); // Memuat semua tiket saat halaman pertama kali dibuka
        });

        function loadTickets() {
            fetch(API_URL)
                .then(response => response.json())
                .then(tickets => {
                    const ticketList = document.getElementById('ticketList');
                    ticketList.innerHTML = '';
                    tickets.forEach(ticket => {
                        ticketList.innerHTML += 
                            `<tr>
                                <td>${ticket.id}</td>
                                <td>${ticket.destination}</td>
                                <td>${ticket.date}</td>
                                <td>${ticket.price}</td>
                                <td>${ticket.stock}</td>
                                <td class="btn-group-action">
                                    <button class="btn btn-sm btn-warning me-1" onclick="editTicket(${ticket.id})">Edit</button>
                                    <button class="btn btn-sm btn-danger" onclick="deleteTicket(${ticket.id})">Hapus</button>
                                </td>
                            </tr>`;
                    });
                })
                .catch(error => alert('Error loading tickets: ' + error));
        }

        function searchTicket() {
            const id = document.getElementById('searchInput').value.trim();
            if (!id) {
                loadTickets(); // Jika input ID kosong, tampilkan semua tiket
                return;
            }

            // URL untuk mencari tiket berdasarkan ID
            const url = `${API_URL}/${id}`;

            fetch(url)
                .then(response => response.json())
                .then(ticket => {
                    const ticketList = document.getElementById('ticketList');
                    ticketList.innerHTML = '';

                    if (ticket.message) {
                        alert('Ticket not found');
                        return;
                    }

                    ticketList.innerHTML = 
                        `<tr>
                            <td>${ticket.id}</td>
                            <td>${ticket.destination}</td>
                            <td>${ticket.date}</td>
                            <td>${ticket.price}</td>
                            <td>${ticket.stock}</td>
                            <td class="btn-group-action">
                                <button class="btn btn-sm btn-warning me-1" onclick="editTicket(${ticket.id})">Edit</button>
                                <button class="btn btn-sm btn-danger" onclick="deleteTicket(${ticket.id})">Hapus</button>
                            </td>
                        </tr>`;
                })
                .catch(error => alert('Error searching ticket: ' + error));
        }

        function editTicket(id) {
            fetch(`${API_URL}/${id}`)
                .then(response => response.json())
                .then(ticket => {
                    document.getElementById('ticketId').value = ticket.id;
                    document.getElementById('destination').value = ticket.destination;
                    document.getElementById('date').value = ticket.date;
                    document.getElementById('price').value = ticket.price;
                    document.getElementById('stock').value = ticket.stock;
                    document.getElementById('modalTitle').textContent = 'Edit Tiket';
                    ticketModal.show();
                })
                .catch(error => alert('Error loading ticket details: ' + error));
        }

        function deleteTicket(id) {
            if (confirm('Are you sure you want to delete this ticket?')) {
                fetch(`${API_URL}/${id}`, { method: 'DELETE' })
                    .then(response => response.json())
                    .then(data => {
                        alert('Ticket deleted successfully');
                        loadTickets();
                    })
                    .catch(error => alert('Error deleting ticket: ' + error));
            }
        }

        function saveTicket() {
            const ticketId = document.getElementById('ticketId').value;
            const ticketData = {
                destination: document.getElementById('destination').value,
                date: document.getElementById('date').value,
                price: document.getElementById('price').value,
                stock: document.getElementById('stock').value
            };

            const method = ticketId ? 'PUT' : 'POST';
            const url = ticketId ? `${API_URL}/${ticketId}` : API_URL;

            fetch(url, {
                method: method,
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify(ticketData)
            })
            .then(response => response.json())
            .then(data => {
                alert(ticketId ? 'Ticket updated successfully' : 'Ticket added successfully');
                ticketModal.hide();
                loadTickets();
                resetForm();
            })
            .catch(error => alert('Error saving ticket: ' + error));
        }

        function resetForm() {
            document.getElementById('ticketId').value = '';
            document.getElementById('ticketForm').reset();
            document.getElementById('modalTitle').textContent = 'Tambah Tiket';
        }

        // Reset form when modal is closed
        document.getElementById('ticketModal').addEventListener('hidden.bs.modal', resetForm);
    </script>
</body>
</html>


```
### tikets_api.php ###
```
<?php
header("Content-Type: application/json; charset=UTF-8");
header("Access-Control-Allow-Origin: *");
header("Access-Control-Allow-Methods: GET, POST, PUT, DELETE");
header("Access-Control-Allow-Headers: Content-Type, Access-Control-Allow-Headers, Authorization, X-Requested-With");

$method = $_SERVER['REQUEST_METHOD'];
$request = [];

if (isset($_SERVER['PATH_INFO'])) {
    $request = explode('/', trim($_SERVER['PATH_INFO'], '/'));
}

function getConnection() {
    $host = 'localhost';
    $db   = 'ticketstore';
    $user = 'root';
    $pass = ''; // Ganti dengan password MySQL Anda jika ada
    $charset = 'utf8mb4';

    $dsn = "mysql:host=$host;dbname=$db;charset=$charset";
    $options = [
        PDO::ATTR_ERRMODE            => PDO::ERRMODE_EXCEPTION,
        PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
        PDO::ATTR_EMULATE_PREPARES   => false,
    ];
    try {
        return new PDO($dsn, $user, $pass, $options);
    } catch (\PDOException $e) {
        throw new \PDOException($e->getMessage(), (int)$e->getCode());
    }
}

function response($status, $data = NULL) {
    header("HTTP/1.1 " . $status);
    if ($data) {
        echo json_encode($data);
    }
    exit();
}

$db = getConnection();

switch ($method) {
    case 'GET':
        if (!empty($request) && isset($request[0])) {
            $id = $request[0];
            $stmt = $db->prepare("SELECT * FROM tickets WHERE id = ?");
            $stmt->execute([$id]);
            $ticket = $stmt->fetch();
            if ($ticket) {
                response(200, $ticket);
            } else {
                response(404, ["message" => "Ticket not found"]);
            }
        } else {
            $stmt = $db->query("SELECT * FROM tickets");
            $tickets = $stmt->fetchAll();
            response(200, $tickets);
        }
        break;
    
    case 'POST':
        $data = json_decode(file_get_contents("php://input"));
        if (!isset($data->destination) || !isset($data->date) || !isset($data->price) || !isset($data->stock)) {
            response(400, ["message" => "Missing required fields"]);
        }
        $sql = "INSERT INTO tickets (destination, date, price, stock) VALUES (?, ?, ?, ?)";
        $stmt = $db->prepare($sql);
        if ($stmt->execute([$data->destination, $data->date, $data->price, $data->stock])) {
            response(201, ["message" => "Ticket created", "id" => $db->lastInsertId()]);
        } else {
            response(500, ["message" => "Failed to create ticket"]);
        }
        break;
    
    case 'PUT':
        if (empty($request) || !isset($request[0])) {
            response(400, ["message" => "Ticket ID is required"]);
        }
        $id = $request[0];
        $data = json_decode(file_get_contents("php://input"));
        if (!isset($data->destination) || !isset($data->date) || !isset($data->price) || !isset($data->stock)) {
            response(400, ["message" => "Missing required fields"]);
        }
        $sql = "UPDATE tickets SET destination = ?, date = ?, price = ?, stock = ? WHERE id = ?";
        $stmt = $db->prepare($sql);
        if ($stmt->execute([$data->destination, $data->date, $data->price, $data->stock, $id])) {
            response(200, ["message" => "Ticket updated"]);
        } else {
            response(500, ["message" => "Failed to update ticket"]);
        }
        break;

    case 'DELETE':
        if (empty($request) || !isset($request[0])) {
            response(400, ["message" => "Ticket ID is required"]);
        }
        $id = $request[0];
        $sql = "DELETE FROM tickets WHERE id = ?";
        $stmt = $db->prepare($sql);
        if ($stmt->execute([$id])) {
            response(200, ["message" => "Ticket deleted"]);
        } else {
            response(500, ["message" => "Failed to delete ticket"]);
        }
        break;
    
    default:
        response(405, ["message" => "Method not allowed"]);
        break;
}
?>

```

