---
layout: kredit
---


  <section id="simulasi" class="mt-0">
    <div class="row justify-content-center">
      <div class="card col-lg-7">
        <div class="card-body">
          <div class="row form">
            <div class="col-md-4">
              <label for="nominal" class="mt-2">Nominal Pinjaman</label>
            </div>
            <div class="col-md-8">
              <div class="input-group mb-3">
                <span class="input-group-text" id="basic-addon1">Rp.</span>
                <input type="number" name="nominal" id="nominal" class="form-control">
              </div>
            </div>
          </div>
          <div class="row form">
            <div class="col-md-4">
              <label for="jkw" class="mt-2">Jangka Waktu</label>
            </div>
            <div class="col-md-8">
              <div class="input-group mb-3">
                <input type="number" id="jkw" name="jkw" class="form-control">
                <span class="input-group-text" id="basic-addon1">Bulan</span>
              </div>
            </div>
          </div>
          <div class="row form">
            <div class="col-md-4">
              <label for="suku_bunga" class="mt-2">Suku Bunga</label>
            </div>
            <div class="col-md-8">
              <div class="input-group mb-3">
                <input type="number" id="suku_bunga" name="suku_bunga" class="form-control" value="12" disabled>
                <span class="input-group-text" id="basic-addon1">%</span>
              </div>
            </div>
          </div>
          <div class="row form">
            <div class="col-md-4">
              <label for="sistem_angsuran" class="mt-2">Sistem Angsuran</label>
            </div>
            <div class="col-md-8">
              <div class="input-group mb-3">
                <select name="sistem_angsuran" class="form-control" id="sistem_angsuran">
                  <option value="flat">Flat</option>
                  <option value="anuitas">Anuitas</option>
                </select>
              </div>
            </div>
          </div>
          <hr>
          <div class="row form">
            <div class="col-md-4"></div>
            <div class="col-md-4">
              <div class="input-group d-grid mb-3">
                <button type="submit" class="btn btn-primary btn-sm">Hitung</button>
              </div>
            </div>
            <div class="col-md-4">
              <div class="input-group d-grid mb-3">
                <button type="reset" class="btn btn-warning btn-sm">Reset</button>
              </div>
            </div>
          </div>
          <hr>
          <div class="row form">
            <table class="table table-bordered table-striped table-responsive">
              <thead>
                <tr>
                  <td><strong>Angsuran Ke</strong></td>
                  <td><strong>Angsuran Pokok</strong></td>
                  <td><strong>Angsuran Bunga</strong></td>
                  <td><strong>Total Angsuran</strong></td>
                  <td><strong>Baki Debet</strong></td>
                </tr>
              </thead>
              <tbody>
                <!-- diisi otomatis dari sript -->
              </tbody>
              <tfoot class="table-success">
                <tr>
                  <td><strong>TOTAL</strong></td>
                  <td id="totalPokok"></td>
                  <td id="totalBunga"></td>
                  <td id="totalAngsuran"></td>
                  <td></td>
                </tr>
              </tfoot>
            </table>
          </div>
        </div>
      </div>
    </div>
  </section>

<script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/js/bootstrap.bundle.min.js"
    integrity="sha384-C6RzsynM9kWDrMNeT87bh95OGNyZPhcTNXj1NW7RuBCsyN/o0jlpcV8Qyq46cDfL"
    crossorigin="anonymous"></script>

  <script>
     function formatRupiah(angka) {
      return new Intl.NumberFormat('id-ID', {
        style: 'currency',
        currency: 'IDR',
        minimumFractionDigits: 0,
        maximumFractionDigits: 0
      }).format(angka);
    }

    document.querySelector("button[type='submit']").addEventListener("click", function (e) {
      e.preventDefault();

      let nominal = parseFloat(document.getElementById("nominal").value);
      let jkw = parseInt(document.getElementById("jkw").value);
      let bungaTahunan = parseFloat(document.getElementById("suku_bunga").value) / 100;
      let sistem = document.getElementById("sistem_angsuran").value;

      if (isNaN(nominal) || isNaN(jkw)) {
        alert("Isi dulu Nominal Pinjaman dan Jangka Waktu!");
        return;
      }

      let tbody = document.querySelector("table tbody");
      tbody.innerHTML = ""; // reset tabel
      let totalPokok = 0, totalBunga = 0, totalAngsuran = 0;

      if (sistem === "flat") {
        let pokok = nominal / jkw;
        let bungaBulanan = nominal * (bungaTahunan / 12);
        let total = pokok + bungaBulanan;
        let sisa = nominal;

        for (let i = 1; i <= jkw; i++) {
          sisa -= pokok;
          totalPokok += pokok;
          totalBunga += bungaBulanan;
          totalAngsuran += total;

          tbody.innerHTML += `
        <tr>
          <td>${i}</td>
          <td>${formatRupiah(Math.round(pokok))}</td>
          <td>${formatRupiah(Math.round(bungaBulanan))}</td>
          <td>${formatRupiah(Math.round(total))}</td>
          <td>${formatRupiah(Math.round(sisa > 0 ? sisa : 0))}</td>
        </tr>
      `;
        }
      } else if (sistem === "anuitas") {
        let i = bungaTahunan / 12;
        let angsuranTetap = nominal * (i / (1 - Math.pow(1 + i, -jkw)));
        let sisa = nominal;

        for (let k = 1; k <= jkw; k++) {
          let bunga = sisa * i;
          let pokok = angsuranTetap - bunga;
          sisa -= pokok;

          totalPokok += pokok;
          totalBunga += bunga;
          totalAngsuran += angsuranTetap;

          tbody.innerHTML += `
        <tr>
          <td>${k}</td>
          <td>${formatRupiah(Math.round(pokok))}</td>
          <td>${formatRupiah(Math.round(bunga))}</td>
          <td>${formatRupiah(Math.round(angsuranTetap))}</td>
          <td>${formatRupiah(Math.round(sisa > 0 ? sisa : 0))}</td>
        </tr>
      `;
        }
      }

      // update total di footer
      document.getElementById("totalPokok").innerText = formatRupiah(Math.round(totalPokok));
      document.getElementById("totalBunga").innerText = formatRupiah(Math.round(totalBunga));
      document.getElementById("totalAngsuran").innerText = formatRupiah(Math.round(totalAngsuran));
    });

    // Reset tombol
    document.querySelector("button[type='reset']").addEventListener("click", function () {
      document.querySelector("table tbody").innerHTML = "";
      document.getElementById("totalPokok").innerText = "";
      document.getElementById("totalBunga").innerText = "";
      document.getElementById("totalAngsuran").innerText = "";
    });
  </script>