using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Mvc.Rendering;
using Microsoft.EntityFrameworkCore;
using ProductModule.Models;

namespace ProductModule.Controllers
{
    public class ProductsController : Controller
    {
        private readonly ProductDbContext _context;

        public ProductsController(ProductDbContext context)
        {
            _context = context;
        }

        // GET: Products
        public async Task<IActionResult> Index()
        {
            var products = await _context.Products
                .Include(p => p.Category)
                .Include(p => p.ProductImages) // Assuming navigation exists
                .ToListAsync();

            return View(products);
        }


        // GET: Products/Details/5
        public async Task<IActionResult> Details(int? id)
        {
            if (id == null)
            {
                return NotFound();
            }

            var product = await _context.Products
                .Include(p => p.Category)
                .FirstOrDefaultAsync(m => m.ProductId == id);
            if (product == null)
            {
                return NotFound();
            }

            return View(product);
        }

        // GET: Products/Create
        public IActionResult Create()
        {
            ViewData["CategoryId"] = new SelectList(_context.Categorys, "CategoryId", "CategoryName");
            return View();
        }

        // POST: Products/Create
        // To protect from overposting attacks, enable the specific properties you want to bind to.
        // For more details, see http://go.microsoft.com/fwlink/?LinkId=317598.
        [HttpPost]
        [ValidateAntiForgeryToken]
        [HttpPost]
        //[ValidateAntiForgeryToken]
        public async Task<IActionResult> Create([Bind("ProductId,Name,Description,Price,CategoryId,IsActive")] Product product, List<IFormFile> images)
        {
            // Ensure at least 2 images
            if (images == null || images.Count < 2)
            {
                ModelState.AddModelError("", "Please upload at least 2 images.");
            }

            //if (ModelState.IsValid)
            //{
                // Save Product first
                _context.Add(product);
                await _context.SaveChangesAsync();

                // Save uploaded images
                foreach (var image in images)
                {
                    if (image != null && image.Length > 0)
                    {
                        var fileName = Guid.NewGuid().ToString() + Path.GetExtension(image.FileName);
                        var filePath = Path.Combine(Directory.GetCurrentDirectory(), "wwwroot/images", fileName);

                        // Save to wwwroot/images
                        using (var stream = new FileStream(filePath, FileMode.Create))
                        {
                            await image.CopyToAsync(stream);
                        }

                        // Add to DB
                        var productImage = new ProductImage
                        {
                            ProductId = product.ProductId,
                            ImageUrl = "/images/" + fileName
                        };

                        _context.ProductImages.Add(productImage);
                    }
                }

                await _context.SaveChangesAsync();
                return RedirectToAction(nameof(Index));
            //}

            // If failed, reload dropdown with names (not IDs)
            ViewData["CategoryId"] = new SelectList(_context.Categorys, "CategoryId", "CategoryName", product.CategoryId);
            return View(product);
        }


        // GET: Products/Edit/5
        public async Task<IActionResult> Edit(int? id)
        {
            if (id == null)
            {
                return NotFound();
            }

            var product = await _context.Products.FindAsync(id);
            if (product == null)
            {
                return NotFound();
            }

            ViewData["CategoryId"] = new SelectList(_context.Categorys, "CategoryId", "CategoryName", product.CategoryId);

            // Optional: pass existing images for preview
            ViewBag.ExistingImages = await _context.ProductImages
                .Where(pi => pi.ProductId == product.ProductId)
                .ToListAsync();

            return View(product);
        }


        // POST: Products/Edit/5
        // To protect from overposting attacks, enable the specific properties you want to bind to.
        // For more details, see http://go.microsoft.com/fwlink/?LinkId=317598.
        [HttpPost]
        [ValidateAntiForgeryToken]
        [HttpPost]
        public async Task<IActionResult> Edit(int id, [Bind("ProductId,Name,Description,Price,CategoryId,IsActive")] Product product, List<IFormFile> images)
        {
            if (id != product.ProductId)
            {
                return NotFound();
            }

            //if (ModelState.IsValid)
            //{
                try
                {
                    // Update product basic fields
                    _context.Update(product);
                    await _context.SaveChangesAsync();

                    // ✅ Handle image upload if 2 or more new images are provided
                    if (images != null && images.Count >= 2)
                    {
                        // (Optional) Delete old images from DB and server
                        var oldImages = _context.ProductImages.Where(pi => pi.ProductId == product.ProductId).ToList();
                        foreach (var old in oldImages)
                        {
                            var oldPath = Path.Combine(Directory.GetCurrentDirectory(), "wwwroot", old.ImageUrl.TrimStart('/'));
                            if (System.IO.File.Exists(oldPath))
                                System.IO.File.Delete(oldPath);
                        }

                        _context.ProductImages.RemoveRange(oldImages);
                        await _context.SaveChangesAsync();

                        // Save new images
                        string uploadPath = Path.Combine(Directory.GetCurrentDirectory(), "wwwroot/uploads");
                        if (!Directory.Exists(uploadPath))
                            Directory.CreateDirectory(uploadPath);

                        foreach (var image in images)
                        {
                            string uniqueFileName = Guid.NewGuid() + Path.GetExtension(image.FileName);
                            string filePath = Path.Combine(uploadPath, uniqueFileName);

                            using (var stream = new FileStream(filePath, FileMode.Create))
                            {
                                await image.CopyToAsync(stream);
                            }

                            var productImage = new ProductImage
                            {
                                ProductId = product.ProductId,  // Ensure your ProductImage model has this FK
                                //ImageName = uniqueFileName,
                                ImageUrl = "/uploads/" + uniqueFileName
                            };
                            _context.ProductImages.Add(productImage);
                        }

                        await _context.SaveChangesAsync();
                    }
                }
                catch (DbUpdateConcurrencyException)
                {
                    if (!ProductExists(product.ProductId))
                    {
                        return NotFound();
                    }
                    else
                    {
                        throw;
                    }
                }

            //}

            ViewData["CategoryId"] = new SelectList(_context.Categorys, "CategoryId", "CategoryName", product.CategoryId);
            return RedirectToAction(nameof(Index));
            //return View(product);
        }

        // GET: Products/Delete/5
        public async Task<IActionResult> Delete(int? id)
        {
            if (id == null)
            {
                return NotFound();
            }

            var product = await _context.Products
                .Include(p => p.Category)
                .FirstOrDefaultAsync(m => m.ProductId == id);
            if (product == null)
            {
                return NotFound();
            }

            return View(product);
        }

        // POST: Products/Delete/5
        [HttpPost, ActionName("Delete")]
        [ValidateAntiForgeryToken]
        public async Task<IActionResult> DeleteConfirmed(int id)
        {
            var product = await _context.Products.FindAsync(id);
            if (product != null)
            {
                _context.Products.Remove(product);
            }

            await _context.SaveChangesAsync();
            return RedirectToAction(nameof(Index));
        }

        private bool ProductExists(int id)
        {
            return _context.Products.Any(e => e.ProductId == id);
        }
    }
}
