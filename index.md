## folder klien indek.php ##


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
