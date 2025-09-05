# CRIAÇAO APP - EVENTPRO
Teste de um aplicativo que é capaz de criar e eventos, sejam eles palestras, shows, corporativos, esportivos, etc. com controle de vendas dos ingressos, local, data e hora.

import React, { useState, useEffect } from "react";
import { Event, Ticket } from "@/entities/all";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card";
import { Badge } from "@/components/ui/badge";
import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue } from "@/components/ui/select";
import { Calendar, MapPin, Users, DollarSign, ShoppingCart, Search } from "lucide-react";
import { motion, AnimatePresence } from "framer-motion";
import { format } from "date-fns";
import { ptBR } from "date-fns/locale";

import EventCard from "../components/tickets/EventCard";
import PurchaseModal from "../components/tickets/PurchaseModal";

const categoryColors = {
  show: "bg-purple-100 text-purple-800 border-purple-200",
  palestra: "bg-blue-100 text-blue-800 border-blue-200",
  esporte: "bg-green-100 text-green-800 border-green-200",
  corporativo: "bg-yellow-100 text-yellow-800 border-yellow-200",
  exposicao: "bg-pink-100 text-pink-800 border-pink-200",
  outro: "bg-gray-100 text-gray-800 border-gray-200"
};

export default function ComprarIngressos() {
  const [events, setEvents] = useState([]);
  const [filteredEvents, setFilteredEvents] = useState([]);
  const [isLoading, setIsLoading] = useState(true);
  const [searchTerm, setSearchTerm] = useState("");
  const [selectedCategory, setSelectedCategory] = useState("all");
  const [selectedEvent, setSelectedEvent] = useState(null);
  const [showPurchaseModal, setShowPurchaseModal] = useState(false);

  useEffect(() => {
    loadEvents();
  }, []);

  useEffect(() => {
    filterEvents();
  }, [events, searchTerm, selectedCategory]);

  const loadEvents = async () => {
    setIsLoading(true);
    try {
      const eventsData = await Event.list("-created_date");
      const activeEvents = (eventsData || []).filter(event => event.status === "ativo");
      setEvents(activeEvents);
    } catch (error) {
      console.error("Erro ao carregar eventos:", error);
      setEvents([]);
    }
    setIsLoading(false);
  };

  const filterEvents = () => {
    let filtered = events;

    if (searchTerm) {
      filtered = filtered.filter(event =>
        event.title.toLowerCase().includes(searchTerm.toLowerCase()) ||
        event.location.toLowerCase().includes(searchTerm.toLowerCase())
      );
    }

    if (selectedCategory !== "all") {
      filtered = filtered.filter(event => event.category === selectedCategory);
    }

    setFilteredEvents(filtered);
  };

  const handlePurchaseClick = (event) => {
    setSelectedEvent(event);
    setShowPurchaseModal(true);
  };

  const handlePurchaseComplete = () => {
    setShowPurchaseModal(false);
    setSelectedEvent(null);
    // Poderia recarregar eventos aqui se necessário
  };

  return (
    <div className="min-h-screen bg-gradient-to-br from-slate-50 via-blue-50 to-purple-50 p-6">
      <div className="max-w-7xl mx-auto">
        <motion.div
          initial={{ opacity: 0, y: -20 }}
          animate={{ opacity: 1, y: 0 }}
          className="mb-8"
        >
          <h1 className="text-3xl md:text-4xl font-bold text-slate-900 mb-2">
            Comprar Ingressos
          </h1>
          <p className="text-slate-600 text-lg">Encontre e compre ingressos para os melhores eventos</p>
        </motion.div>

        {/* Filtros */}
        <motion.div
          initial={{ opacity: 0, y: 20 }}
          animate={{ opacity: 1, y: 0 }}
          transition={{ delay: 0.1 }}
          className="mb-8"
        >
          <Card className="shadow-lg border-0 bg-white/80 backdrop-blur-sm">
            <CardContent className="p-6">
              <div className="flex flex-col md:flex-row gap-4">
                <div className="flex-1">
                  <div className="relative">
                    <Search className="absolute left-3 top-1/2 transform -translate-y-1/2 text-slate-400 w-4 h-4" />
                    <Input
                      placeholder="Buscar eventos ou locais..."
                      value={searchTerm}
                      onChange={(e) => setSearchTerm(e.target.value)}
                      className="pl-10 border-slate-300 focus:border-blue-500"
                    />
                  </div>
                </div>
                <div className="w-full md:w-48">
                  <Select value={selectedCategory} onValueChange={setSelectedCategory}>
                    <SelectTrigger className="border-slate-300 focus:border-blue-500">
                      <SelectValue placeholder="Categoria" />
                    </SelectTrigger>
                    <SelectContent>
                      <SelectItem value="all">Todas as categorias</SelectItem>
                      <SelectItem value="show">Shows</SelectItem>
                      <SelectItem value="palestra">Palestras</SelectItem>
                      <SelectItem value="esporte">Esportes</SelectItem>
                      <SelectItem value="corporativo">Corporativos</SelectItem>
                      <SelectItem value="exposicao">Exposições</SelectItem>
                      <SelectItem value="outro">Outros</SelectItem>
                    </SelectContent>
                  </Select>
                </div>
              </div>
            </CardContent>
          </Card>
        </motion.div>

        {/* Lista de Eventos */}
        <div className="grid md:grid-cols-2 lg:grid-cols-3 gap-6">
          <AnimatePresence>
            {isLoading ? (
              Array(6).fill(0).map((_, index) => (
                <motion.div
                  key={index}
                  initial={{ opacity: 0, y: 20 }}
                  animate={{ opacity: 1, y: 0 }}
                  transition={{ delay: index * 0.1 }}
                  className="animate-pulse"
                >
                  <Card className="shadow-lg border-0 bg-white/80 backdrop-blur-sm">
                    <div className="h-48 bg-slate-200 rounded-t-lg"></div>
                    <CardContent className="p-6">
                      <div className="h-4 bg-slate-200 rounded mb-2"></div>
                      <div className="h-4 bg-slate-200 rounded w-2/3 mb-4"></div>
                      <div className="h-3 bg-slate-200 rounded mb-2"></div>
                      <div className="h-3 bg-slate-200 rounded w-1/2"></div>
                    </CardContent>
                  </Card>
                </motion.div>
              ))
            ) : (
              filteredEvents.map((event, index) => (
                <EventCard
                  key={event.id}
                  event={event}
                  index={index}
                  categoryColors={categoryColors}
                  onPurchaseClick={handlePurchaseClick}
                />
              ))
            )}
          </AnimatePresence>
        </div>

        {!isLoading && filteredEvents.length === 0 && (
          <motion.div
            initial={{ opacity: 0 }}
            animate={{ opacity: 1 }}
            className="text-center py-16"
          >
            <ShoppingCart className="w-16 h-16 text-slate-300 mx-auto mb-4" />
            <h3 className="text-xl font-semibold text-slate-900 mb-2">
              Nenhum evento encontrado
            </h3>
            <p className="text-slate-500">
              Tente ajustar os filtros ou verifique novamente em breve
            </p>
          </motion.div>
        )}

        {/* Modal de Compra */}
        {showPurchaseModal && selectedEvent && (
          <PurchaseModal
            event={selectedEvent}
            isOpen={showPurchaseModal}
            onClose={() => setShowPurchaseModal(false)}
            onComplete={handlePurchaseComplete}
          />
        )}
      </div>
    </div>
  );
}
