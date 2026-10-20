import { useState, useEffect, useMemo } from 'react'
import './App.css'
const API = 'http://127.0.0.1:8000'
const emptyForm = { name: '', description: '', price: '', quantity: '' }

function App() {
  const [products, setProducts] = useState([])
  const [form, setForm] = useState(emptyForm)
  const [editingId, setEditingId] = useState(null)
  const [currentImage, setCurrentImage] = useState('')
  const [selectedImage, setSelectedImage] = useState(null)
  const [preview, setPreview] = useState('')
  const [message, setMessage] = useState('')
  const [error, setError] = useState('')
  const [loading, setLoading] = useState(false)

  async function loadProducts() {
    try {
      const response = await fetch(`${API}/products`)
      if (!response.ok) throw new Error()
      setProducts(await response.json())
      setError('')
    } catch {
      setError('Connexion impossible. Vérifiez que le backend fonctionne.')
    }
  }

  useEffect(() => {
    loadProducts()
  }, [])

  useEffect(() => {
    if (!selectedImage) {
      setPreview('')
      return
    }
    const url = URL.createObjectURL(selectedImage)
    setPreview(url)
    return () => URL.revokeObjectURL(url)
  }, [selectedImage])

  function handleChange(event) {
    setForm({ ...form, [event.target.name]: event.target.value })
  }

  function chooseImage(event) {
    const file = event.target.files?.[0]
    if (!file) return

    const allowed = ['image/jpeg', 'image/png', 'image/webp', 'image/gif']
    if (!allowed.includes(file.type)) {
      setError('Choisissez une image JPG, PNG, WEBP ou GIF.')
      event.target.value = ''
      return
    }
    if (file.size > 5 * 1024 * 1024) {
      setError('La taille maximale de l’image est de 5 Mo.')
      event.target.value = ''
      return
    }

    setError('')
    setSelectedImage(file)
    setMessage('')
  }

  function resetForm() {
    setForm(emptyForm)
    setEditingId(null)
    setCurrentImage('')
    setSelectedImage(null)
    setMessage('')
    setError('')
  }

  async function handleSubmit(event) {
    event.preventDefault()
    setLoading(true)
    setMessage('')
    setError('')

    try {
      const payload = {
        name: form.name.trim(),
        description: form.description.trim(),
        price: Number(form.price),
        quantity: Number(form.quantity),
      }

      const response = await fetch(
        editingId ? `${API}/products/${editingId}` : `${API}/products`,
        {
          method: editingId ? 'PUT' : 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify(payload),
        },
      )

      if (!response.ok) {
        const result = await response.json()
        const detail = result.detail
        throw new Error(Array.isArray(detail) ? detail[0]?.msg : detail || 'Erreur lors de l’enregistrement.')
      }

      const product = await response.json()

      if (selectedImage) {
        const data = new FormData()
        data.append('image', selectedImage)

        const imageResponse = await fetch(`${API}/products/${product.id}/image`, {
          method: 'POST',
          body: data,
        })

        if (!imageResponse.ok) {
          const result = await imageResponse.json()
          throw new Error(result.detail || 'Le produit est enregistré, mais le téléversement de l’image a échoué.')
        }
      }

      setMessage(editingId ? 'Produit modifié avec succès !' : 'Produit ajouté avec succès !')
      resetForm()
      await loadProducts()
    } catch (err) {
      setError(err.message || 'Erreur de connexion au serveur.')
    } finally {
      setLoading(false)
    }
  }

  function editProduct(product) {
    setEditingId(product.id)
    setForm({
      name: product.name,
      description: product.description || '',
      price: String(product.price),
      quantity: String(product.quantity),
    })
    setCurrentImage(product.image_url ? `${API}${product.image_url}` : '')
    setSelectedImage(null)
    setPreview('')
    setMessage('')
    setError('')
    window.scrollTo({ top: 0, behavior: 'smooth' })
  }

  async function deleteProduct(product) {
    if (!window.confirm(`Supprimer « ${product.name} » ?`)) return

    try {
      const response = await fetch(`${API}/products/${product.id}`, { method: 'DELETE' })
      if (!response.ok) throw new Error('Suppression impossible.')
      setMessage('Produit supprimé avec succès.')
      await loadProducts()
    } catch (err) {
      setError(err.message)
    }
  }

  const imageToShow = preview || currentImage

  return (
    <main className="container">
      <header className="hero">
        <div className="hero-icon">📦</div>
        <p className="eyebrow">APPLICATION CRUD · FULL STACK</p>
        <h1>Gestion des produits</h1>
        <p className="subtitle">Gérez votre inventaire facilement depuis une seule interface.</p>
        <div className="stats">
          <span className="stat-number">{products.length}</span>
          <span className="stat-label">produit(s) enregistré(s)</span>
        </div>
      </header>

      <section className="panel">
        <div className="section-heading">
          <div>
            <p className="eyebrow">FORMULAIRE</p>
            <h2>{editingId ? 'Modifier le produit' : 'Ajouter un produit'}</h2>
          </div>
          {editingId && <span className="edit-badge">Modification</span>}
        </div>

        <form onSubmit={handleSubmit}>
          <div className="form-grid">
            <label>
              Nom du produit *
              <input name="name" value={form.name} onChange={handleChange}
                placeholder="Ex. Ordinateur portable" maxLength="120" required />
            </label>

            <label>
              Prix (DT) *
              <input name="price" type="number" min="0" step="0.01"
                value={form.price} onChange={handleChange} placeholder="Ex. 1250.00" required />
            </label>

            <label>
              Quantité *
              <input name="quantity" type="number" min="0" step="1"
                value={form.quantity} onChange={handleChange} placeholder="Ex. 10" required />
            </label>

            <label className="description-field">
              Description
              <textarea name="description" value={form.description} onChange={handleChange}
                placeholder="Description du produit..." rows="3" />
            </label>
          </div>

          <div className="upload-section">
            <div className="upload-heading">
              <div>
                <p className="eyebrow">PHOTO DU PRODUIT</p>
                <h3>Upload Image</h3>
                <p className="upload-hint">Ajoutez une belle photo pour présenter votre produit.</p>
              </div>
              <span className="image-icon">✦</span>
            </div>

            <div className={`image-dropzone ${imageToShow ? 'has-image' : ''}`}>
              {imageToShow ? (
                <>
                  <img className="image-preview" src={imageToShow} alt="Aperçu du produit" />
                  <div className="image-overlay">
                    <span>Aperçu de votre image</span>
                  </div>
                </>
              ) : (
                <div className="upload-placeholder">
                  <div className="upload-symbol">↑</div>
                  <h4>Déposez votre image ici</h4>
                  <p>ou choisissez une image depuis votre ordinateur</p>
                  <span className="file-types">JPG · PNG · WEBP · GIF — 5 Mo maximum</span>
                </div>
              )}
            </div>

            <div className="upload-actions">
              <label className="upload-button">
                <span>↑</span> {imageToShow ? 'Changer l’image' : 'Choisir une image'}
                <input
                  type="file"
                  accept="image/jpeg,image/png,image/webp,image/gif"
                  onChange={chooseImage}
                  hidden
                />
              </label>
              {imageToShow && (
                <button type="button" className="remove-image-button" onClick={() => {
                  setSelectedImage(null)
                  setPreview('')
                  setCurrentImage('')
                }}>
                  Retirer l’image
                </button>
              )}
            </div>
            {editingId && currentImage && !selectedImage && (
              <p className="upload-hint">L’image actuelle sera conservée si vous ne la remplacez pas.</p>
            )}
          </div>

          <div className="form-actions">
            <button className="primary-button" type="submit" disabled={loading}>
              {loading ? 'Enregistrement...' : editingId ? 'Enregistrer les modifications' : '+ Ajouter le produit'}
            </button>
            {editingId && (
              <button className="secondary-button" type="button" onClick={resetForm}>Annuler</button>
            )}
          </div>
        </form>

        {message && <p className="notice success">{message}</p>}
        {error && <p className="notice error">{error}</p>}
      </section>

      <section className="panel products-panel">
        <div className="section-heading">
          <div>
            <p className="eyebrow">INVENTAIRE</p>
            <h2>Liste des produits</h2>
          </div>
          <button className="secondary-button" onClick={loadProducts} type="button">↻ Actualiser</button>
        </div>

        {products.length === 0 ? (
          <div className="empty-state">
            <span>🗃️</span>
            <h3>Aucun produit pour le moment</h3>
            <p>Ajoutez votre premier produit avec le formulaire ci-dessus.</p>
          </div>
        ) : (
          <div className="table-wrapper">
            <table>
              <thead>
                <tr><th>Photo</th><th>ID</th><th>Produit</th><th>Prix</th><th>Quantité</th><th>Actions</th></tr>
              </thead>
              <tbody>
                {products.map((product) => (
                  <tr key={product.id}>
                    <td>
                      {product.image_url ? (
                        <img className="product-thumbnail" src={`${API}${product.image_url}`} alt={product.name} />
                      ) : <span className="no-thumbnail">📦</span>}
                    </td>
                    <td className="id-cell">#{product.id}</td>
                    <td>
                      <strong>{product.name}</strong>
                      <p className="description-text">{product.description || 'Aucune description'}</p>
                    </td>
                    <td className="price-cell">{Number(product.price).toFixed(2)} DT</td>
                    <td><span className="quantity-badge">{product.quantity}</span></td>
                    <td>
                      <div className="row-actions">
                        <button className="edit-button" onClick={() => editProduct(product)}>Modifier</button>
                        <button className="delete-button" onClick={() => deleteProduct(product)}>Supprimer</button>
                      </div>
                    </td>
                  </tr>
                ))}
              </tbody>
            </table>
          </div>
        )}
      </section>

      <footer>React · FastAPI · MySQL — TP1 CRUD Full Stack</footer>
    </main>
  )
}

export default App
