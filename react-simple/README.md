このハンズオンを進めるためには [`react-simple`](.) フォルダに移動する必要があります。

# React Simple - SNS App 実装ガイド

このハンズオンを始める前に、 [`react-simple`](.) フォルダをVSCodeで開いてください。

## プロジェクト概要

このプロジェクトは、React を使用したソーシャルネットワーク（掲示板）アプリケーションの実装ガイドです。
まずダミーデータを使用してReactの基本的な機能を実装し、その後にバックエンドAPI（localhost:9999）と連携して完全な動作を実現します。

## 完成イメージ

- 投稿の一覧表示とページネーション
- 新規投稿の作成
- 投稿への いいね 機能
- コメントの表示と追加（モーダル）
- 投稿の削除
- ダークモード対応

## 前提条件

- Node.js がインストールされていること
- React の基本的な知識があること
- JSX、useState、useEffect の基本的な使い方を知っていること

## 学習目標

- React Hooks を使用した状態管理
- コンポーネント間のデータフロー
- イベントハンドリングとフォーム処理
- 条件付きレンダリング
- API連携の基本

## 実装ステップ

### Step 1: プロジェクトの準備と環境構築

**目標**: 基本的なVite+React環境を準備し、必要な依存関係をインストール

#### 1.0 ターミナルを立ち上げる

[ターミナルの立ち上げ方](../README.md#-vscode-でのターミナル操作)を参考に、VSCodeでターミナルを立ち上げてください。

#### 1.1 プロジェクトの初期化確認

```bash
# 現在のディレクトリで依存関係を確認
npm install
```

#### 1.2 開発サーバーの起動確認

```bash
# 開発サーバーを起動
npm run dev
```

#### 1.3 基本構造の確認

Reactで重要なファイルを確認します.

- `src/main.jsx` - React アプリケーションの読み込みポイント（Cのmain関数のようなもの）
- `src/App.jsx` - メインアプリケーションコンポーネント（実際のアプリケーションが動いているコンポーネント）
- `src/utils/` - ユーティリティ関数（日付フォーマット、ローカルストレージ）

### Step 2: ダミーデータの作成

**目標**: API連携前に使用するダミーデータを作成

#### 2.1 dataディレクトリの作成

```bash
mkdir -p src/data
```

#### 2.2 ダミーデータの実装

`src/data/dummyData.js` を作成し、以下のソースコードを貼り付けて保存して下さい。

```javascript
// ダミーデータの管理
let nextId = 4;

export const dummyPosts = [
  {
    id: 1,
    author: 'Alice',
    content: 'こんにちは！今日はいい天気ですね。',
    created_at: new Date(Date.now() - 3600000).toISOString(), // 1時間前
    like_count: 3,
    comment_count: 2,
    comments: [
      {
        id: 1,
        author: 'Bob',
        content: 'そうですね！散歩日和です。',
        created_at: new Date(Date.now() - 1800000).toISOString(),
      },
      {
        id: 2,
        author: 'Charlie',
        content: '私も外に出かけたいです。',
        created_at: new Date(Date.now() - 900000).toISOString(),
      },
    ],
  },
  {
    id: 2,
    author: 'Bob',
    content: 'Reactの勉強をしています。難しいですが楽しいです！',
    created_at: new Date(Date.now() - 7200000).toISOString(), // 2時間前
    like_count: 5,
    comment_count: 1,
    comments: [
      {
        id: 3,
        author: 'Alice',
        content: 'がんばって！応援してます。',
        created_at: new Date(Date.now() - 3600000).toISOString(),
      },
    ],
  },
  {
    id: 3,
    author: 'Charlie',
    content: 'みなさんこんばんは！今日も一日お疲れ様でした。',
    created_at: new Date(Date.now() - 10800000).toISOString(), // 3時間前
    like_count: 2,
    comment_count: 0,
    comments: [],
  },
];

export function addPost(author, content) {
  const newPost = {
    id: nextId++,
    author,
    content,
    created_at: new Date().toISOString(),
    like_count: 0,
    comment_count: 0,
    comments: [],
  };
  dummyPosts.unshift(newPost);
  return newPost;
}

export function deletePost(postId) {
  const index = dummyPosts.findIndex(post => post.id === parseInt(postId));
  if (index !== -1) {
    dummyPosts.splice(index, 1);
    return true;
  }
  return false;
}

export function toggleLike(postId, increment = true) {
  const post = dummyPosts.find(post => post.id === parseInt(postId));
  if (post) {
    post.like_count += increment ? 1 : -1;
    post.like_count = Math.max(0, post.like_count);
  }
}

export function addComment(postId, author, content) {
  const post = dummyPosts.find(post => post.id === parseInt(postId));
  if (post) {
    const newComment = {
      id: Date.now(),
      author,
      content,
      created_at: new Date().toISOString(),
    };
    post.comments.push(newComment);
    post.comment_count = post.comments.length;
    return newComment;
  }
  return null;
}

export function getPaginatedPosts(page = 1, limit = 10) {
  const startIndex = (page - 1) * limit;
  const endIndex = startIndex + limit;
  const paginatedPosts = dummyPosts.slice(startIndex, endIndex);

  return {
    posts: paginatedPosts,
    pagination: {
      page,
      limit,
      totalCount: dummyPosts.length,
      totalPages: Math.ceil(dummyPosts.length / limit),
      hasNext: endIndex < dummyPosts.length,
      hasPrev: page > 1,
    },
  };
}

export function getPostById(postId) {
  return dummyPosts.find(post => post.id === parseInt(postId));
}
```

### Step 3: 基本コンポーネントの実装

**目標**: SNSの投稿表示に必要な基本コンポーネントを作成していきます.

#### 3.1 componentsディレクトリの作成

フォルダ位置：[react-simple](.)であることを確認

```bash
mkdir -p src/components
```

#### 3.2 Postコンポーネントの実装

１つ１つの投稿を表示するコンポーネントを実装します。

`src/components/Post.jsx` を作成：

```jsx
import { formatDate } from '../utils/date.js';

function Post({ post, currentUser, onLike, onComment, onDelete }) {
  // サーバーからisLikedが提供されている場合はそれを使用、なければfalse
  const liked = post.isLiked || false;

  return (
    <div className="post">
      <div className="post-header">
        <span className="post-author">{post.author}</span>
        <span className="post-date">{formatDate(post.created_at)}</span>
      </div>
      <div className="post-content">{post.content}</div>
      <div className="post-actions">
        <button
          className={`like-btn ${liked ? 'liked' : ''}`}
          onClick={() => onLike(post.id)}
        >
          <span className="like-icon">{liked ? '❤️' : '🤍'}</span>
          <span className="like-count">{post.like_count || 0}</span>
        </button>
        <button className="comment-btn" onClick={() => onComment(post.id)}>
          💬 {post.comment_count || 0}
        </button>
        {post.author === currentUser && (
          <button className="delete-btn" onClick={() => onDelete(post.id)}>
            🗑️ 削除
          </button>
        )}
      </div>
    </div>
  );
}

export default Post;
```

#### 3.3 PostListコンポーネントの実装

投稿をリスト形式で表示するコンポーネントを作ります。

`src/components/PostList.jsx` を作成：

```jsx
import Post from './Post';

function PostList({
  posts,
  currentUser,
  onLike,
  onComment,
  onDelete,
  onPageChange,
}) {

  if (!posts || posts.length === 0) {
    return <div className="empty">投稿がありません。</div>;
  }

  return (
    <div className="posts-container">
      <h2>タイムライン</h2>
      <div className="posts-list">
        {posts.map(post => (
          <Post
            key={post.id}
            post={post}
            currentUser={currentUser}
            onLike={onLike}
            onComment={onComment}
            onDelete={onDelete}
          />
        ))}
      </div>
    </div>
  );
}

export default PostList;
```

### Step 4: 投稿作成機能の実装

**目標**: 新規投稿を作成するフォームコンポーネントを実装

#### 4.1 PostFormコンポーネントの実装

`src/components/PostForm.jsx` を作成：

```jsx
import { useState } from 'react';

function PostForm({ currentUser, onSubmit }) {
  const [content, setContent] = useState('');
  const [isSubmitting, setIsSubmitting] = useState(false);

  const handleSubmit = async e => {
    e.preventDefault();

    if (!content.trim()) {
      alert('投稿内容を入力してください。');
      return;
    }

    setIsSubmitting(true);
    try {
      await onSubmit(content.trim());
      setContent('');
    } catch (error) {
      alert('投稿の作成に失敗しました。');
    } finally {
      setIsSubmitting(false);
    }
  };

  const handleKeyPress = e => {
    if (e.key === 'Enter' && e.ctrlKey) {
      handleSubmit(e);
    }
  };

  return (
    <div className="new-post-container">
      <h2>新しい投稿</h2>
      <form onSubmit={handleSubmit}>
        <textarea
          value={content}
          onChange={e => setContent(e.target.value)}
          onKeyPress={handleKeyPress}
          placeholder="今何してる？"
          rows={4}
        />
        <button type="submit">投稿する</button>
      </form>
    </div>
  );
}

export default PostForm;
```

### Step 5: メインアプリケーションの状態管理

**目標**: App.jsx で全体の状態を管理し、コンポーネント間のデータフローを実装

#### 5.1 App.jsxの状態管理実装

`src/App.jsx` を更新：

```jsx
import { useState, useEffect } from 'react';
import './App.css';
import PostForm from './components/PostForm';
import PostList from './components/PostList';
import {
  getPosts,
  createPost,
  deletePost,
  addLike,
  removeLike,
} from './services/postService';

function App() {
  const [posts, setPosts] = useState([]);
  const [currentUser, setCurrentUser] = useState('ゲスト');
  const [isLoading, setIsLoading] = useState(false);
  const [error, setError] = useState(null);
  const [commentModalOpen, setCommentModalOpen] = useState(false);
  const [selectedPostId, setSelectedPostId] = useState(null);
  const [pagination, setPagination] = useState({
    page: 1,
    limit: 10,
    totalCount: 0,
    totalPages: 0,
    hasNext: false,
    hasPrev: false,
  });

  useEffect(() => {
    loadPosts(1);
  }, []);

  const loadPosts = async (page = pagination.page) => {
    try {
      setIsLoading(true);
      setError(null);
      const data = await getPosts(page, pagination.limit, currentUser);
      setPosts(data.posts);
      setPagination(data.pagination);
    } catch (err) {
      console.error('投稿の取得に失敗しました:', err);
      setError('投稿の取得に失敗しました');
    } finally {
      setIsLoading(false);
    }
  };

  const handleCreatePost = async (author, content) => {
    await createPost(author, content);
    await loadPosts(1);
  };

  const handleDeletePost = async postId => {
    if (!confirm('この投稿を削除しますか？')) {
      return;
    }

    try {
      await deletePost(postId);
      await loadPosts();
    } catch (error) {
      console.error('投稿の削除に失敗しました:', error);
      alert('投稿の削除に失敗しました。');
    }
  };

  const handleToggleLike = async postId => {
    const post = posts.find(p => p.id === postId);
    const liked = post?.isLiked || false;

    try {
      if (liked) {
        await removeLike(postId, currentUser);
      } else {
        await addLike(postId, currentUser);
      }

      await loadPosts();
    } catch (error) {
      console.error('いいねの更新に失敗しました:', error);
    }
  };

  const handleOpenComments = postId => {
    setSelectedPostId(postId);
    setCommentModalOpen(true);
  };

  const handleCloseComments = () => {
    setCommentModalOpen(false);
    setSelectedPostId(null);
  };

  const handleCommentAdded = async () => {
    await loadPosts();
  };

  const handlePageChange = page => {
    loadPosts(page);
  };

  return (
    <div className="app">
      <header>
        <h1>SNS掲示板</h1>
        <div className="user-info">
          <label>ユーザー名: </label>
          <input
            type="text"
            value={currentUser}
            onChange={e => setCurrentUser(e.target.value)}
            placeholder="名前を入力"
          />
        </div>
      </header>

      <main>
        <PostForm currentUser={currentUser} onSubmit={handleCreatePost} />

        <PostList
          posts={posts}
          currentUser={currentUser}
          onLike={handleToggleLike}
          onComment={handleOpenComments}
          onDelete={handleDeletePost}
          isLoading={isLoading}
          error={error}
          pagination={pagination}
          onPageChange={handlePageChange}
        />
      </main>
    </div>
  );
}

export default App;
```

### Step 6: ページネーション機能の実装

**目標**: 投稿一覧のページネーション機能を実装

#### 6.1 Paginationコンポーネントの実装

`src/components/Pagination.jsx` を作成：

```jsx
import './Pagination.css';

function Pagination({ pagination, onPageChange }) {
  const { page, totalPages, hasNext, hasPrev, totalCount } = pagination;

  if (totalPages <= 1) {
    return null;
  }

  const handlePageClick = newPage => {
    if (newPage >= 1 && newPage <= totalPages) {
      onPageChange(newPage);
    }
  };

  const getPageNumbers = () => {
    const pages = [];
    const maxVisible = 5;
    let start = Math.max(1, page - Math.floor(maxVisible / 2));
    let end = Math.min(totalPages, start + maxVisible - 1);

    if (end - start + 1 < maxVisible) {
      start = Math.max(1, end - maxVisible + 1);
    }

    for (let i = start; i <= end; i++) {
      pages.push(i);
    }

    return pages;
  };

  return (
    <div className="pagination">
      <div className="pagination-info">
        {totalCount}件中 {(page - 1) * pagination.limit + 1}-
        {Math.min(page * pagination.limit, totalCount)}件を表示
      </div>

      <div className="pagination-controls">
        <button
          onClick={() => handlePageClick(page - 1)}
          disabled={!hasPrev}
          className="pagination-button"
        >
          前へ
        </button>

        {getPageNumbers().map(pageNum => (
          <button
            key={pageNum}
            onClick={() => handlePageClick(pageNum)}
            className={`pagination-button ${page === pageNum ? 'active' : ''}`}
          >
            {pageNum}
          </button>
        ))}

        <button
          onClick={() => handlePageClick(page + 1)}
          disabled={!hasNext}
          className="pagination-button"
        >
          次へ
        </button>
      </div>
    </div>
  );
}

export default Pagination;

```

#### 6.2 PostList.jsxでのPaginationコンポーネント使用

PostList.jsxを以下のように変更して下さい。

```diff
import Post from './Post';
+ import Pagination

function PostList({
  posts,
  currentUser,
  onLike,
  onComment,
  onDelete,
  onPageChange,
+ pagination
}) {

  if (!posts || posts.length === 0) {
    return <div className="empty">投稿がありません。</div>;
  }

  return (
    <div className="posts-container">
      <h2>タイムライン</h2>
      <div className="posts-list">
        {posts.map(post => (
          <Post
            key={post.id}
            post={post}
            currentUser={currentUser}
            onLike={onLike}
            onComment={onComment}
            onDelete={onDelete}
          />
        ))}
      </div>
+      {pagination && (
+       <Pagination pagination={pagination} onPageChange={onPageChange} />
+      )}
    </div>
  );
}

export default PostList;
```

### Step 7: コメント機能の実装

**目標**: モーダルウィンドウでのコメント表示・追加機能を実装

#### 7.1 Commentコンポーネントの実装

`src/components/Comment.jsx` を作成：

```jsx
import { formatDate } from '../utils/date.js';

function Comment({ comment }) {
  return (
    <div className="comment">
      <div className="comment-header">
        <span className="comment-author">{comment.author}</span>
        <span className="comment-date">{formatDate(comment.created_at)}</span>
      </div>
      <div className="comment-content">{comment.content}</div>
    </div>
  );
}

export default Comment;
```

#### 7.2 CommentModalコンポーネントの実装

`src/components/CommentModal.jsx` を作成：

```jsx
import { useState } from 'react';
import Comment from './Comment.jsx';
import { formatDate } from '../utils/date.js';

function CommentModal({ post, currentUser, onClose, onAddComment }) {
  const [commentContent, setCommentContent] = useState('');
  const [isSubmitting, setIsSubmitting] = useState(false);

  const handleSubmit = async e => {
    e.preventDefault();

    if (!commentContent.trim()) {
      alert('コメントを入力してください。');
      return;
    }

    setIsSubmitting(true);
    try {
      await onAddComment(post.id, commentContent.trim());
      setCommentContent('');
    } catch (error) {
      alert('コメントの追加に失敗しました。');
    } finally {
      setIsSubmitting(false);
    }
  };

  const handleKeyPress = e => {
    if (e.key === 'Enter' && e.ctrlKey) {
      handleSubmit(e);
    }
  };

  return (
    <div className="comment-modal" onClick={onClose}>
      <div className="modal-content" onClick={e => e.stopPropagation()}>
        <div className="modal-header">
          <h3>コメント</h3>
          <button className="close-btn" onClick={onClose}>
            ×
          </button>
        </div>

        <div className="modal-body">
          <div className="original-post">
            <div className="post-header">
              <span className="post-author">{post.author}</span>
              <span className="post-date">{formatDate(post.created_at)}</span>
            </div>
            <div className="post-content">{post.content}</div>
          </div>

          <div className="comments-section">
            <h4>コメント ({post.comments.length})</h4>
            <div className="comments-list">
              {post.comments.length === 0 ? (
                <p className="no-comments">コメントはまだありません。</p>
              ) : (
                post.comments.map(comment => (
                  <Comment key={comment.id} comment={comment} />
                ))
              )}
            </div>
          </div>

          <form className="comment-form" onSubmit={handleSubmit}>
            <textarea
              value={commentContent}
              onChange={e => setCommentContent(e.target.value)}
              onKeyPress={handleKeyPress}
              placeholder="コメントを入力（Ctrl+Enterで投稿）"
              rows="3"
              disabled={isSubmitting}
            />
            <div className="form-actions">
              <span className="user-info">投稿者: {currentUser}</span>
              <button
                type="submit"
                disabled={isSubmitting || !commentContent.trim()}
              >
                {isSubmitting ? '投稿中...' : 'コメントする'}
              </button>
            </div>
          </form>
        </div>
      </div>
    </div>
  );
}

export default CommentModal;
```

### Step 8: 削除機能の実装

**目標**: 投稿の削除機能を実装

削除機能は既にStep 5で実装済みです。以下のポイントを確認：

- 投稿者本人のみが削除ボタンを表示
- 削除前に確認ダイアログを表示
- 削除後に投稿一覧を更新

### Step 9: スタイリングの実装

**目標**: アプリケーションのスタイリングを完成させる

#### 9.1 App.cssの実装

`src/App.css` を更新：

```css
/* CSS変数の定義 */
:root {
  --primary-color: #1976d2;
  --secondary-color: #666;
  --success-color: #4caf50;
  --danger-color: #f44336;
  --warning-color: #ff9800;
  --light-color: #f5f5f5;
  --dark-color: #333;
  --border-color: #ddd;
  --text-color: #333;
  --bg-color: #fff;
  --shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  --border-radius: 8px;
}

/* ダークモード */
@media (prefers-color-scheme: dark) {
  :root {
    --text-color: #f5f5f5;
    --bg-color: #1a1a1a;
    --light-color: #2a2a2a;
    --border-color: #444;
  }
}

/* 基本スタイル */
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family:
    -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
  line-height: 1.6;
  color: var(--text-color);
  background-color: var(--bg-color);
}

.app {
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
}

/* ヘッダー */
.app-header {
  background: var(--light-color);
  padding: 20px;
  border-radius: var(--border-radius);
  margin-bottom: 20px;
  box-shadow: var(--shadow);
}

.app-header h1 {
  color: var(--primary-color);
  margin-bottom: 10px;
}

.user-controls {
  display: flex;
  align-items: center;
  gap: 10px;
}

.user-controls input {
  padding: 8px 12px;
  border: 1px solid var(--border-color);
  border-radius: 4px;
  background: var(--bg-color);
  color: var(--text-color);
}

/* 投稿フォーム */
.post-form {
  background: var(--light-color);
  padding: 20px;
  border-radius: var(--border-radius);
  margin-bottom: 20px;
  box-shadow: var(--shadow);
}

.post-form h2 {
  color: var(--primary-color);
  margin-bottom: 15px;
}

.post-form textarea {
  width: 100%;
  padding: 12px;
  border: 1px solid var(--border-color);
  border-radius: 4px;
  background: var(--bg-color);
  color: var(--text-color);
  resize: vertical;
  font-family: inherit;
  margin-bottom: 10px;
}

.form-actions {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.user-info {
  font-size: 0.9em;
  color: var(--secondary-color);
}

.post-form button {
  background: var(--primary-color);
  color: white;
  border: none;
  padding: 10px 20px;
  border-radius: 4px;
  cursor: pointer;
  font-size: 16px;
  transition: background-color 0.2s;
}

.post-form button:hover:not(:disabled) {
  background: #1565c0;
}

.post-form button:disabled {
  background: var(--secondary-color);
  cursor: not-allowed;
}

/* 投稿セクション */
.posts-section h2 {
  color: var(--primary-color);
  margin-bottom: 15px;
}

.posts-list {
  display: flex;
  flex-direction: column;
  gap: 15px;
}

.post {
  background: var(--light-color);
  border: 1px solid var(--border-color);
  border-radius: var(--border-radius);
  padding: 20px;
  box-shadow: var(--shadow);
}

.post-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 10px;
}

.post-author {
  font-weight: bold;
  color: var(--primary-color);
}

.post-date {
  color: var(--secondary-color);
  font-size: 0.9em;
}

.post-content {
  margin-bottom: 15px;
  line-height: 1.6;
  white-space: pre-wrap;
}

.post-actions {
  display: flex;
  gap: 10px;
}

.post-actions button {
  border: none;
  padding: 8px 12px;
  border-radius: 4px;
  cursor: pointer;
  font-size: 14px;
  transition: all 0.2s;
  display: flex;
  align-items: center;
  gap: 4px;
}

.like-btn {
  background: var(--bg-color);
  color: var(--text-color);
  border: 1px solid var(--border-color);
}

.like-btn.liked {
  background: #ffeaa7;
  color: #d63031;
  border-color: #d63031;
}

.comment-btn {
  background: var(--bg-color);
  color: var(--text-color);
  border: 1px solid var(--border-color);
}

.comment-btn:hover {
  background: var(--primary-color);
  color: white;
}

.delete-btn {
  background: var(--danger-color);
  color: white;
}

.delete-btn:hover {
  background: #d32f2f;
}

/* ページネーション */
.pagination {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-top: 20px;
  padding: 20px;
  background: var(--light-color);
  border-radius: var(--border-radius);
  box-shadow: var(--shadow);
}

.pagination-info {
  font-size: 0.9em;
  color: var(--secondary-color);
}

.pagination-controls {
  display: flex;
  gap: 5px;
}

.pagination-btn {
  padding: 8px 12px;
  border: 1px solid var(--border-color);
  background: var(--bg-color);
  color: var(--text-color);
  cursor: pointer;
  border-radius: 4px;
  transition: all 0.2s;
}

.pagination-btn:hover:not(:disabled) {
  background: var(--primary-color);
  color: white;
}

.pagination-btn.active {
  background: var(--primary-color);
  color: white;
}

.pagination-btn:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

/* モーダル */
.comment-modal {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}

.modal-content {
  background: var(--bg-color);
  border-radius: var(--border-radius);
  width: 90%;
  max-width: 600px;
  max-height: 80vh;
  overflow-y: auto;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.2);
}

.modal-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 20px;
  border-bottom: 1px solid var(--border-color);
}

.modal-header h3 {
  color: var(--primary-color);
}

.close-btn {
  background: none;
  border: none;
  font-size: 24px;
  cursor: pointer;
  color: var(--secondary-color);
  padding: 0;
  width: 30px;
  height: 30px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.close-btn:hover {
  color: var(--danger-color);
}

.modal-body {
  padding: 20px;
}

.original-post {
  background: var(--light-color);
  padding: 15px;
  border-radius: var(--border-radius);
  margin-bottom: 20px;
}

.comments-section h4 {
  color: var(--primary-color);
  margin-bottom: 10px;
}

.comments-list {
  max-height: 300px;
  overflow-y: auto;
  margin-bottom: 20px;
}

.comment {
  background: var(--light-color);
  padding: 15px;
  border-radius: var(--border-radius);
  margin-bottom: 10px;
  border-left: 3px solid var(--primary-color);
}

.comment-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 8px;
}

.comment-author {
  font-weight: bold;
  color: var(--primary-color);
}

.comment-date {
  font-size: 0.8em;
  color: var(--secondary-color);
}

.comment-content {
  line-height: 1.5;
  white-space: pre-wrap;
}

.no-comments {
  text-align: center;
  color: var(--secondary-color);
  font-style: italic;
  padding: 20px;
}

.comment-form {
  border-top: 1px solid var(--border-color);
  padding-top: 20px;
}

.comment-form textarea {
  width: 100%;
  padding: 10px;
  border: 1px solid var(--border-color);
  border-radius: 4px;
  background: var(--bg-color);
  color: var(--text-color);
  resize: vertical;
  font-family: inherit;
  margin-bottom: 10px;
}

/* 状態表示 */
.loading,
.error,
.empty {
  text-align: center;
  padding: 40px;
  color: var(--secondary-color);
}

.error {
  color: var(--danger-color);
}

/* レスポンシブデザイン */
@media (max-width: 768px) {
  .app {
    padding: 10px;
  }

  .post-header {
    flex-direction: column;
    align-items: flex-start;
    gap: 5px;
  }

  .post-actions {
    flex-wrap: wrap;
  }

  .pagination {
    flex-direction: column;
    gap: 10px;
  }

  .pagination-controls {
    flex-wrap: wrap;
    justify-content: center;
  }

  .modal-content {
    width: 95%;
    margin: 10px;
  }

  .form-actions {
    flex-direction: column;
    gap: 10px;
    align-items: flex-start;
  }
}
```

### Step 10: エラーハンドリングと UX 向上

**目標**: エラーハンドリングとユーザーエクスペリエンスを向上させる

#### 10.1 エラーハンドリングの実装

既にStep 5で基本的なエラーハンドリングを実装済み。以下を確認：

- try-catch文によるエラー処理
- ユーザーフレンドリーなエラーメッセージ
- ローディング状態の表示

#### 10.2 UX向上のポイント

- キーボードショートカット（Ctrl+Enter）
- 無効な操作の防止（空の投稿など）
- 即座のフィードバック（いいね状態の即時反映）

### Step 11: API連携への移行

**目標**: ダミーデータからAPI連携に切り替え

#### 11.1 バックエンドAPI起動

```bash
# 別ターミナルで
npm run dev:backend
```

#### 11.2 API サービス層の実装

`src/services/postService.js` を作成：

```javascript
import { apiGet, apiPost, apiDelete } from '../utils/api.js';

export async function getPosts(page = 1, limit = 10) {
  return apiGet(`/posts?page=${page}&limit=${limit}`);
}

export async function getPost(postId) {
  return apiGet(`/posts/${postId}`);
}

export async function createPost(author, content) {
  return apiPost('/posts', { author, content });
}

export async function deletePost(postId) {
  return apiDelete(`/posts/${postId}`);
}

export async function addLike(postId, userId) {
  return apiPost(`/posts/${postId}/likes`, { userId });
}

export async function removeLike(postId, userId) {
  return apiDelete(`/posts/${postId}/likes`, { userId });
}

export async function addComment(postId, author, content) {
  return apiPost(`/posts/${postId}/comments`, { author, content });
}
```

#### 11.3 App.jsxの修正

ダミーデータの関数をpostServiceの関数に置き換え：

```jsx
// import文を変更
import {
  getPosts,
  getPost,
  createPost,
  deletePost,
  addLike,
  removeLike,
  addComment,
} from './services/postService.js';

// 関数をasync/awaitに変更
const loadPosts = async (page = currentPage) => {
  setLoading(true);
  setError(null);

  try {
    const data = await getPosts(page, pagination.limit);
    setPosts(data.posts);
    setPagination(data.pagination);
    setCurrentPage(page);
  } catch (err) {
    setError('投稿の読み込みに失敗しました。');
  } finally {
    setLoading(false);
  }
};

// 他の関数も同様に修正...
```

## 技術的なポイント

### React Hooks

- **useState**: コンポーネントの状態管理
- **useEffect**: 副作用の処理（データ取得など）
- **カスタムフック**: 共通ロジックの抽出

### コンポーネント設計

- **単一責任の原則**: 各コンポーネントが一つの責任を持つ
- **Props**: 親から子へのデータ受け渡し
- **イベントハンドリング**: 子から親への情報伝達

### 状態管理

- **リフトアップ**: 状態を共通の親コンポーネントに移動
- **状態の正規化**: 重複を避けた状態設計
- **派生状態**: 既存の状態から計算される値

### パフォーマンス最適化

- **条件付きレンダリング**: 必要な時のみコンポーネントを描画
- **キーの適切な使用**: リストレンダリングの最適化
- **不要な再レンダリングの防止**: React.memo、useMemo、useCallback

## 学習のポイント

### React の基本概念

- **コンポーネント**: UIの再利用可能な部品
- **JSX**: JavaScriptの拡張構文
- **仮想DOM**: 効率的なDOM更新

### 実践的な開発スキル

- **状態管理**: アプリケーションの状態を適切に管理
- **イベント処理**: ユーザーインタラクションの処理
- **API連携**: 外部サービスとの通信

### 現代的な開発手法

- **関数コンポーネント**: クラスコンポーネントより簡潔
- **Hooks**: 状態管理と副作用の処理
- **ES6+**: モダンなJavaScript機能の活用

このガイドに従って段階的に実装することで、React を使用した本格的なWebアプリケーションの開発スキルを身につけることができます。

[← プロジェクトトップに戻る](../README.md)
